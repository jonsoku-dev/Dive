# Dive 데이터 흐름 상세 분석

이 문서는 Dive 애플리케이션에서의 데이터 흐름 과정을 상세히 설명합니다. 사용자 입력부터 AI 응답 출력까지 전체적인 데이터 처리 과정과 각 단계에서의 데이터 변환 및 컴포넌트 상호작용을 이해할 수 있습니다.

## 목차

- [전체 데이터 흐름 개요](#전체-데이터-흐름-개요)
- [사용자 입력 처리 흐름](#사용자-입력-처리-흐름)
- [LLM 응답 처리 흐름](#llm-응답-처리-흐름)
- [도구 호출 흐름](#도구-호출-흐름)
- [데이터 저장 흐름](#데이터-저장-흐름)
- [상태 관리 흐름](#상태-관리-흐름)
- [커스터마이징 포인트](#커스터마이징-포인트)

## 전체 데이터 흐름 개요

Dive 애플리케이션의 데이터 흐름은 다음과 같은 주요 단계로 구성됩니다:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  사용자 입력 │ ──> │ 프론트엔드 UI │ ──> │ 백엔드 서비스 │ ──> │   LLM 처리  │ ──> │  응답 생성  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                  │                    │
                                                                  ▼                    ▼
                                                           ┌─────────────┐     ┌─────────────┐
                                                           │  도구 호출  │ ──> │  도구 실행  │
                                                           └─────────────┘     └─────────────┘
                                                                                      │
                                                                                      ▼
┌─────────────┐     ┌─────────────┐                             ┌─────────────┐
│ 응답 표시   │ <── │ 데이터 저장  │ <─────────────────────────── │ 결과 처리   │
└─────────────┘     └─────────────┘                             └─────────────┘
```

## 사용자 입력 처리 흐름

### 1. 사용자 입력 캡처
**파일**: `/src/views/Chat/ChatInput.tsx`

사용자가 메시지를 입력하고 전송 버튼을 클릭하거나 단축키(Enter)를 누르면 입력이 캡처됩니다.

```typescript
// 사용자 입력 처리 예시 코드
const handleSendMessage = useCallback(() => {
  if (message.trim() || selectedFiles.length > 0) {
    onSendMessage(message, selectedFiles);
    setMessage("");
    setSelectedFiles([]);
  }
}, [message, selectedFiles, onSendMessage]);
```

### 2. 메시지 이벤트 전파
**파일**: `/src/views/Chat/index.tsx`

캡처된 메시지는 채팅 인터페이스에서 처리되며, API 요청으로 변환됩니다.

```typescript
const onSendMsg = useCallback(async (msg: string, files?: FileList) => {
  if (isChatStreaming) return;

  const formData = new FormData();
  if (msg)
    formData.append("message", msg);

  if (currentChatId.current)
    formData.append("chatId", currentChatId.current);

  if (files) {
    Array.from(files).forEach(file => {
      formData.append("files", file);
    });
  }

  // UI 상태 업데이트
  setMessages(prev => [...prev, userMessage, aiMessage]);
  setIsChatStreaming(true);
  
  // API 요청 처리
  handlePost(formData, "formData", "/api/chat");
}, [isChatStreaming, scrollToBottom]);
```

### 3. 백엔드 요청 처리
**파일**: `/services/routes/chat.ts`

프론트엔드 요청은 Express 라우터에 의해 처리됩니다.

```typescript
// 채팅 API 요청 처리
router.post("/", async (req, res) => {
  try {
    const message = req.body.message;
    const chatId = req.body.chatId || randomUUID();
    const files = req.files?.files;

    // 스트리밍 응답 설정
    res.setHeader("Content-Type", "text/event-stream");
    res.setHeader("Cache-Control", "no-cache");
    res.setHeader("Connection", "keep-alive");

    // 쿼리 처리 및 스트리밍
    client.processQuery(
      chatId,
      { text: message, files: files },
      (chunk) => {
        res.write(`data: ${chunk}\n\n`);
      }
    );
    
  } catch (error) {
    // 오류 처리
    res.write(`data: ${JSON.stringify({ error: error.message })}\n\n`);
  }
});
```

### 4. MCP 클라이언트 처리
**파일**: `/services/client.ts`

MCPClient 클래스는 실제 LLM 요청을 처리합니다.

```typescript
public async processQuery(
  chatId: string | undefined,
  input: string | iQueryInput,
  onStream?: (text: string) => void,
  regenerateMessageId?: string,
  fingerprint?: string,
  user_access_token?: string
) {
  // 채팅 ID 및 컨텍스트 설정
  let chat_id = chatId || randomUUID();
  let history: BaseMessage[] = [];
  
  // 시스템 프롬프트 추가
  const systemPrompt = PromptManager.getInstance().getPrompt("system");
  if (systemPrompt) {
    history.push(new SystemMessage(systemPrompt));
  }
  
  // 이전 메시지 로드
  const messageHistory = await getChatWithMessages(chat_id);
  if (messageHistory && messageHistory.messages.length > 0) {
    // 히스토리 처리
    history = await processHistoryMessages(messageHistory.messages, history);
  }
  
  // 도구 및 모델 설정
  const serverManager = MCPServerManager.getInstance();
  const toolClientMap = serverManager.getToolToServerMap();
  const availableTools = serverManager.getAvailableTools();
  
  // 쿼리 처리 및 응답 스트리밍
  const { result, tokenUsage } = await handleProcessQuery(
    toolClientMap,
    availableTools,
    ModelManager.getInstance().getModel(),
    input,
    history,
    onStream,
    chat_id
  );
  
  // 데이터베이스에 저장...
}
```

## LLM 응답 처리 흐름

### 1. LLM 요청 생성 및 전송
**파일**: `/services/processQuery.ts`

사용자 입력과 대화 컨텍스트가 처리되어 LLM에 전송됩니다.

```typescript
// LLM 스트리밍 요청
const stream = await runModel.stream(messages, {
  signal: chatId ? abortControllerMap.get(chatId)?.signal : undefined,
});

// 응답 스트리밍 처리
for await (const chunk of stream) {
  // 토큰 사용량 계산
  caculateTokenUsage(tokenUsage, chunk, currentModelSettings!);
  
  // 텍스트 청크 처리
  if (chunk.content) {
    let chunkMessage = "";
    if (Array.isArray(chunk.content)) {
      // Anthropic 응답 형식 지원
      const textContent = chunk.content.find((item) => item.type === "text" || item.type === "text_delta");
      chunkMessage = textContent?.text || "";
    } else {
      chunkMessage = chunk.content;
    }
    
    currentContent += chunkMessage;
    onStream?.(
      JSON.stringify({
        type: "text",
        content: chunkMessage,
      } as iStreamMessage)
    );
  }
  
  // 도구 호출 처리...
}
```

### 2. 스트리밍 응답 처리
**파일**: `/src/views/Chat/index.tsx`

스트리밍되는 응답 청크를 처리하여 UI에 표시합니다.

```typescript
const handlePost = useCallback(async (body: any, type: "json" | "formData", url: string) => {
  // API 요청...
  
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  
  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    
    const chunk = decoder.decode(value);
    const lines = (chunkBuf + chunk).split("\n");
    
    for (const line of lines) {
      if (line.trim() === "" || !line.startsWith("data: ")) continue;
      
      const dataStr = line.slice(5);
      if (dataStr.trim() === "[DONE]") break;
      
      // 응답 처리
      try {
        const dataObj = JSON.parse(dataStr);
        const data = JSON.parse(dataObj.message);
        
        switch (data.type) {
          case "text":
            // 텍스트 업데이트
            currentText += data.content;
            setMessages(prev => {
              const newMessages = [...prev];
              newMessages[newMessages.length - 1].text = currentText;
              return newMessages;
            });
            break;
            
          case "tool_calls":
            // 도구 호출 시각화
            // ...
            break;
            
          case "tool_result":
            // 도구 결과 표시
            // ...
            break;
            
          // 기타 메시지 타입 처리...
        }
      } catch (error) {
        console.warn(error);
      }
    }
  }
}, []);
```

## 도구 호출 흐름

### 1. 도구 호출 감지
**파일**: `/services/processQuery.ts`

LLM 응답에서 도구 호출을 감지하고 처리합니다.

```typescript
// 도구 호출 감지
if (chunk.tool_calls || chunk.tool_call_chunks || 
    (Array.isArray(chunk.content) && chunk.content.some((item) => item.type === "tool_use"))) {
  let toolCallChunks: any[] = [];
  toolCallChunks = chunk.tool_call_chunks || [];
  
  // 도구 호출 정보 수집
  for (const chunks of toolCallChunks) {
    let index = chunks.index;
    // 기존 도구 호출이 없으면 생성
    if (index !== undefined && index >= 0 && !toolCalls[index]) {
      toolCalls[index] = {
        id: chunks.id,
        type: "function",
        function: {
          name: chunks.name,
          arguments: "",
        },
      };
    }
    
    // 인수 누적
    if (index !== undefined && index >= 0) {
      if (chunks.name) {
        toolCalls[index].function.name = chunks.name;
      }
      
      if (chunks.args || chunks.input) {
        const newArgs = chunks.args || chunks.input || "";
        toolCalls[index].function.arguments += newArgs;
      }
    }
  }
}
```

### 2. 도구 호출 실행
**파일**: `/services/processQuery.ts`

수집된 도구 호출 정보를 기반으로 적절한 MCP 서버로 요청을 라우팅합니다.

```typescript
// 사용자에게 도구 호출 정보 전송
onStream?.(
  JSON.stringify({
    type: "tool_calls",
    content: toolCalls.map((call) => ({
      name: call.function.name,
      arguments: JSON.parse(call.function.arguments || "{}"),
    })),
  } as iStreamMessage)
);

// 병렬로 모든 도구 호출 실행
const toolResults = await Promise.all(
  toolCalls.map(async (toolCall) => {
    try {
      const toolName = toolCall.function.name;
      const toolArgs = JSON.parse(toolCall.function.arguments || "{}");
      const client = toolToClientMap.get(toolName);
      
      // 도구 실행 및 결과 수집
      const result = await client?.callTool(
        {
          name: toolName,
          arguments: toolArgs,
        },
        undefined,
        {
          signal: abortController.signal,
          timeout: 99999000,
        }
      );
      
      // 사용자에게 도구 결과 전송
      onStream?.(
        JSON.stringify({
          type: "tool_result",
          content: {
            name: toolName,
            result: result,
          },
        } as iStreamMessage)
      );
      
      return {
        tool_call_id: toolCall.id,
        role: "tool" as const,
        content: JSON.stringify(result),
      };
    } catch (error) {
      // 오류 처리
    }
  })
);
```

### 3. 도구 결과 처리 및 LLM 응답 계속
**파일**: `/services/processQuery.ts`

도구 실행 결과를 LLM에 다시 전달하여 최종 응답을 생성합니다.

```typescript
// 도구 실행 결과를 대화 컨텍스트에 추가
if (toolResults.length > 0) {
  messages.push(...toolResults.map((result) => new ToolMessage(result)));
}

// 추가 응답이 필요한 경우 다시 LLM에 요청
if (toolCalls.length > 0) {
  // hasToolCalls = true이므로 루프 계속 (다시 LLM 호출)
} else {
  // 최종 응답 전달
  hasToolCalls = false;
}
```

## 데이터 저장 흐름

### 1. 채팅 및 메시지 저장
**파일**: `/services/client.ts`

대화가 완료되면 채팅 및 메시지 데이터가 데이터베이스에 저장됩니다.

```typescript
// 채팅 존재 여부 확인 및 생성
const isChatExists = await checkChatExists(chat_id);
if (!isChatExists) {
  await createChat(chat_id, title || "New Chat", {
    fingerprint: fingerprint,
    user_access_token: user_access_token,
  });
}

// 메시지 저장
const userMessageId = randomUUID();
await createMessage(
  {
    role: "user",
    chatId: chat_id,
    messageId: userMessageId,
    content: userInput || "",
    files: files,
    createdAt: new Date().toISOString(),
  },
  {
    fingerprint: fingerprint,
    user_access_token: user_access_token,
  }
);

const assistantMessageId = randomUUID();
await createMessage(
  {
    role: "assistant",
    chatId: chat_id,
    messageId: assistantMessageId,
    content: result,
    files: [],
    createdAt: new Date().toISOString(),
  },
  {
    LLM_Model: {
      model: ModelManager.getInstance().currentModelSettings?.model || "",
      total_input_tokens: tokenUsage.totalInputTokens,
      total_output_tokens: tokenUsage.totalOutputTokens,
      total_run_time: totalRunTime,
    },
    fingerprint: fingerprint,
    user_access_token: user_access_token,
  }
);
```

### 2. 데이터베이스 상호작용
**파일**: `/services/database/index.ts`

데이터베이스 연산을 처리합니다.

```typescript
// 채팅 생성 예시
export async function createChat(
  chatId: string,
  title: string,
  options?: {
    fingerprint?: string;
    user_access_token?: string;
  }
) {
  const timestamp = new Date().toISOString();
  
  if (dbMode === DatabaseMode.DIRECT) {
    // 직접 데이터베이스 접근
    return await db
      .insert(chats)
      .values({
        id: chatId,
        title,
        createdAt: timestamp,
        updatedAt: timestamp,
        metadata: JSON.stringify({
          fingerprint: options?.fingerprint || null,
          user_access_token: options?.user_access_token || null,
        }),
      })
      .returning()
      .get();
  } else {
    // API 통한 데이터베이스 접근
    const response = await axios.post(`${apiUrl}/chats`, {
      id: chatId,
      title,
      createdAt: timestamp,
      updatedAt: timestamp,
      metadata: {
        fingerprint: options?.fingerprint || null,
        user_access_token: options?.user_access_token || null,
      },
    });
    
    return response.data;
  }
}
```

## 상태 관리 흐름

### 1. Jotai Atoms
**파일**: `/src/atoms/*.ts`

애플리케이션 상태는 Jotai atoms로 관리됩니다.

```typescript
// 채팅 상태 atom 예시
export const currentChatIdAtom = atom<string | null>(null);
export const isChatStreamingAtom = atom<boolean>(false);
export const lastMessageAtom = atom<string>("");
```

### 2. 상태 사용 및 업데이트
**파일**: `/src/views/Chat/index.tsx`

컴포넌트에서 상태를 사용하고 업데이트합니다.

```typescript
// 상태 사용 예시
const [isChatStreaming, setIsChatStreaming] = useAtom(isChatStreamingAtom);
const setCurrentChatId = useSetAtom(currentChatIdAtom);
const setLastMessage = useSetAtom(lastMessageAtom);

// 상태 업데이트 예시
useEffect(() => {
  if (messages.length > 0 && !isChatStreaming) {
    setLastMessage(messages[messages.length - 1].text);
  }
}, [messages, setLastMessage, isChatStreaming]);

useEffect(() => {
  if (chatId && chatId !== currentChatId.current) {
    loadChat(chatId);
    setCurrentChatId(chatId);
  }
}, [chatId, loadChat, setCurrentChatId]);
```

## 커스터마이징 포인트

### 1. 입력 처리 확장
커스텀 입력 형식을 지원하려면 다음 파일을 수정하세요:
- `/src/views/Chat/ChatInput.tsx`: 입력 UI 확장
- `/services/utils/types.ts`: 새 입력 타입 정의 추가
- `/services/client.ts`: 입력 처리 로직 확장

예시:
```typescript
// 새 입력 타입 정의
export interface iCustomQueryInput extends iQueryInput {
  customData?: {
    metadata: Record<string, any>;
    tags: string[];
  };
}

// 입력 처리 로직 확장
if (typeof input === "object" && "customData" in input) {
  // 커스텀 데이터 처리 로직
}
```

### 2. 응답 처리 확장
커스텀 응답 형식을 지원하려면 다음 파일을 수정하세요:
- `/services/processQuery.ts`: 응답 처리 로직 확장
- `/src/views/Chat/index.tsx`: 응답 표시 로직 수정

예시:
```typescript
// 새 응답 타입 추가
case "custom_response":
  // 커스텀 응답 처리
  handleCustomResponse(data.content);
  break;
```

### 3. 도구 통합 확장
새로운 도구 유형을 지원하려면 다음 파일을 수정하세요:
- `/services/mcpServer/index.ts`: 도구 서버 연결 로직 확장
- `/services/utils/toolHandler.ts`: 도구 변환 및 처리 로직 확장

예시:
```typescript
// 커스텀 도구 변환 로직
export function convertCustomTools(tools: any[]): ToolDefinition[] {
  return tools.map((tool) => ({
    function: {
      name: tool.name,
      description: tool.description,
      parameters: transformCustomParameters(tool.parameters),
    },
  }));
}
```

### 4. 저장 메커니즘 확장
데이터 저장 메커니즘을 확장하려면 다음 파일을 수정하세요:
- `/services/database/schema.ts`: 스키마 정의 확장
- `/services/database/index.ts`: 데이터베이스 연산 확장

예시:
```typescript
// 새 테이블 정의
export const customEntities = sqliteTable("custom_entities", {
  id: text("id").primaryKey(),
  chatId: text("chatId")
    .notNull()
    .references(() => chats.id, { onDelete: "cascade" }),
  type: text("type").notNull(),
  data: text("data", { mode: "json" }),
  createdAt: text("createdAt").notNull(),
});

// 새 데이터베이스 연산
export async function createCustomEntity(data: {
  id: string;
  chatId: string;
  type: string;
  data: any;
}) {
  // 구현...
}
```

### 5. UI 커스터마이징
UI를 확장하려면 다음 파일을 수정하세요:
- `/src/components/`: 새 UI 컴포넌트 추가
- `/src/styles/`: 스타일 확장
- `/src/views/`: 새 화면 추가

예시:
```typescript
// 새 UI 컴포넌트
export const CustomToolPanel = () => {
  // 구현...
};

// 새 화면 추가
export const CustomDashboard = () => {
  // 구현...
};
```

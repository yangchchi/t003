# 技术规格（TS）：spec-1778591172025

## 元信息

- 流水线：AI焦虑解压器-生产流水线
- 规格 ID：spec-1778591172025
- 关联 PRD：prd-1778574033401
- 状态：draft
- 更新时间：2026-05-12T13:12:54.108Z


---

# TS 正文（Markdown）

# TS

## 1. 技术栈
- 语言：TypeScript 5+
- 框架：Next.js (React) + Node.js (Express 风格 API Routes)

## 2. 数据模型

### 2.1 前端状态模型 (domain/)
```typescript
// domain/scenario.ts
export type ScenarioId = string;
export type StepId = string;
export type UserInputKey = string;

export interface ScenarioTemplate {
  id: ScenarioId;
  title: string;
  description: string;
  totalSteps: number;
}

export interface Step {
  id: StepId;
  scenarioId: ScenarioId;
  stepNumber: number;
  title: string;
  componentType: 'select' | 'text' | 'multi-select';
  options?: Array<{ label: string; value: string }>;
}

export interface UserInputState {
  scenarioId: ScenarioId;
  currentStep: number;
  inputs: Record<UserInputKey, string | string[]>;
  lastUpdated: number; // timestamp
}
```

### 2.2 API 数据模型 (dto/)
```typescript
// dto/ai-assistant.ts
export interface AIAssistantRequest {
  question: string;
  scenarioId: ScenarioId;
  currentStep?: number;
  userId?: string; // 匿名会话ID
}

export interface AIAssistantResponse {
  answer: string;
  suggestedStep?: number;
  error?: boolean;
}

// dto/scenario.ts
export interface ScenarioListResponse {
  scenarios: ScenarioTemplate[];
}

export interface StepDetailResponse {
  step: Step;
  savedInput?: string | string[];
}
```

## 3. API

### 3.1 GET /api/scenarios
- 路径：`/api/scenarios`
- 请求：无
- 响应：
```typescript
{
  "scenarios": [
    {
      "id": "email_writing",
      "title": "邮件撰写",
      "description": "快速生成专业邮件",
      "totalSteps": 3
    }
  ]
}
```
- 错误码：`500` - 服务端错误

### 3.2 GET /api/scenarios/:scenarioId/steps/:stepNumber
- 路径：`/api/scenarios/:scenarioId/steps/:stepNumber`
- 请求：无
- 响应：
```typescript
{
  "step": {
    "id": "email_step_1",
    "scenarioId": "email_writing",
    "stepNumber": 1,
    "title": "选择邮件目的",
    "componentType": "select",
    "options": [
      { "label": "合作邀约", "value": "collaboration" },
      { "label": "投诉反馈", "value": "complaint" }
    ]
  },
  "savedInput": "collaboration" // 可选
}
```
- 错误码：
  - `404` - 场景或步骤不存在
  - `500` - 服务端错误

### 3.3 POST /api/scenarios/:scenarioId/steps/:stepNumber/input
- 路径：`/api/scenarios/:scenarioId/steps/:stepNumber/input`
- 请求：
```typescript
{
  "input": string | string[],
  "sessionId": string // 前端生成的匿名会话ID
}
```
- 响应：
```typescript
{
  "success": boolean,
  "nextStepAvailable": boolean
}
```
- 错误码：
  - `400` - 输入验证失败
  - `404` - 场景或步骤不存在
  - `500` - 服务端错误

### 3.4 POST /api/ai-assistant/ask
- 路径：`/api/ai-assistant/ask`
- 请求：
```typescript
{
  "question": "第二步的背景信息怎么填？",
  "scenarioId": "email_writing",
  "currentStep": 2,
  "userId": "anonymous_123"
}
```
- 响应：
```typescript
{
  "answer": "背景信息应填写...",
  "suggestedStep": 2
}
// 或错误降级
{
  "answer": "助手暂时休息中，请查看引导步骤内的详细说明。",
  "error": true
}
```
- 错误码：
  - `400` - 请求格式错误
  - `429` - 请求频率限制
  - `500` - AI 服务异常
  - `504` - 请求超时（3秒）

## 4. 核心流程

### 4.1 步骤化引导引擎 (前端伪代码)
```typescript
// components/StepWizard.tsx
const StepWizard: React.FC<StepWizardProps> = ({ scenarioId }) => {
  const [currentStep, setCurrentStep] = useState<number>(1);
  const [userInputs, setUserInputs] = useState<Record<number, any>>({});
  const [isLoading, setIsLoading] = useState(false);
  
  // 加载步骤数据
  const loadStep = async (stepNumber: number) => {
    setIsLoading(true);
    try {
      const response = await fetch(`/api/scenarios/${scenarioId}/steps/${stepNumber}`);
      const data: StepDetailResponse = await response.json();
      
      // 恢复已保存的输入
      if (data.savedInput) {
        setUserInputs(prev => ({ ...prev, [stepNumber]: data.savedInput }));
      }
      
      return data.step;
    } finally {
      setIsLoading(false);
    }
  };
  
  // 处理下一步
  const handleNext = async (currentInput: any) => {
    // 保存当前步骤输入
    await saveInput(currentStep, currentInput);
    
    if (currentStep < totalSteps) {
      // 加载下一个步骤
      const nextStep = currentStep + 1;
      await loadStep(nextStep);
      setCurrentStep(nextStep);
    } else {
      // 跳转到成果页
      window.location.href = `/scenarios/${scenarioId}/result`;
    }
  };
  
  // 处理上一步
  const handlePrev = async () => {
    if (currentStep > 1) {
      const prevStep = currentStep - 1;
      await loadStep(prevStep);
      setCurrentStep(prevStep);
    }
  };
  
  // 保存输入到后端
  const saveInput = async (stepNumber: number, input: any) => {
    await fetch(`/api/scenarios/${scenarioId}/steps/${stepNumber}/input`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        input,
        sessionId: getSessionId()
      })
    });
    
    // 同时更新本地状态
    setUserInputs(prev => ({ ...prev, [stepNumber]: input }));
  };
  
  return (
    <div>
      {/* 进度条 */}
      <ProgressBar current={currentStep} total={totalSteps} />
      
      {/* 步骤内容 */}
      {!isLoading && (
        <StepComponent
          step={currentStepData}
          value={userInputs[currentStep]}
          onChange={(value) => saveInput(currentStep, value)}
        />
      )}
      
      {/* 导航按钮 */}
      <button onClick={handlePrev} disabled={currentStep === 1}>上一步</button>
      <button onClick={() => handleNext(userInputs[currentStep])}>
        {currentStep === totalSteps ? '生成结果' : '下一步'}
      </button>
    </div>
  );
};
```

### 4.2 成果生成逻辑 (后端伪代码)
```typescript
// services/result-generator.ts
export const generatePrompt = (
  scenarioId: ScenarioId,
  inputs: Record<number, any>
): string => {
  const template = getScenarioTemplate(scenarioId);
  
  switch (scenarioId) {
    case 'email_writing':
      const purpose = inputs[1];
      const recipient = inputs[2];
      const tone = inputs[3];
      
      return `请你扮演一位专业销售，给${recipient}写一封${tone}的${purpose}邮件...`;
      
    default:
      throw new Error(`Unknown scenario: ${scenarioId}`);
  }
};

// 预设的模拟结果
export const getMockResult = (scenarioId: ScenarioId): string => {
  const mockResults: Record<ScenarioId, string> = {
    'email_writing': `尊敬的XX公司采购部：\n\n我们诚挚地邀请贵公司...\n\n此致\n敬礼\n[您的姓名]`,
    // 其他场景的模拟结果
  };
  
  return mockResults[scenarioId] || 'AI生成结果将显示在这里';
};
```

## 5. 异常处理

### 5.1 错误规则
```typescript
// infra/error-handler.ts
export class AppError extends Error {
  constructor(
    public code: string,
    public statusCode: number,
    message: string,
    public details?: unknown
  ) {
    super(message);
  }
}

// 定义错误类型
export const ERRORS = {
  // 4xx 客户端错误
  VALIDATION_FAILED: new AppError('VALIDATION_FAILED', 400, '输入验证失败'),
  SCENARIO_NOT_FOUND: new AppError('SCENARIO_NOT_FOUND', 404, '场景不存在'),
  STEP_NOT_FOUND: new AppError('STEP_NOT_FOUND', 404, '步骤不存在'),
  
  // 5xx 服务端错误
  AI_SERVICE_UNAVAILABLE: new AppError('AI_SERVICE_UNAVAILABLE', 500, 'AI服务暂时不可用'),
  AI_SERVICE_TIMEOUT: new AppError('AI_SERVICE_TIMEOUT', 504, 'AI服务响应超时'),
  DATABASE_ERROR: new AppError('DATABASE_ERROR', 500, '数据库操作失败'),
  
  // 业务逻辑错误
  INVALID_STEP_TRANSITION: new AppError('INVALID_STEP_TRANSITION', 400, '无效的步骤跳转'),
  SESSION_EXPIRED: new AppError('SESSION_EXPIRED', 401, '会话已过期'),
} as const;

// 全局错误处理中间件
export const errorHandler = (
  error: unknown,
  req: Request,
  res: Response
) => {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      error: error.code,
      message: error.message,
      details: error.details
    });
  }
  
  // 未知错误
  console.error('Unhandled error:', error);
  return res.status(500).json({
    error: 'INTERNAL_SERVER_ERROR',
    message: '服务器内部错误'
  });
};
```

## 6. 测试用例

### 6.1 单元测试 (services/result-generator.test.ts)
```typescript
import { generatePrompt, getMockResult } from './result-generator';

describe('Result Generator', () => {
  describe('generatePrompt', () => {
    test('正常情况 - 邮件撰写场景', () => {
      const inputs = {
        1: '合作邀约',
        2: 'XX公司采购部',
        3: '正式'
      };
      
      const result = generatePrompt('email_writing', inputs);
      
      expect(result).toBe(
        '请你扮演一位专业销售，给XX公司采购部写一封正式的合作邀约邮件...'
      );
    });
    
    test('异常情况 - 未知场景', () => {
      expect(() => {
        generatePrompt('unknown_scenario', {});
      }).toThrow('Unknown scenario: unknown_scenario');
    });
    
    test('边界情况 - 空输入', () => {
      const inputs = {
        1: '',
        2: '',
        3: ''
      };
      
      const result = generatePrompt('email_writing', inputs);
      
      expect(result).toContain('写一封的邮件');
    });
  });
  
  describe('getMockResult', () => {
    test('正常情况 - 返回预设结果', () => {
      const result = getMockResult('email_writing');
      
      expect(result).toContain('尊敬的XX公司采购部');
      expect(result).toContain('此致\n敬礼');
    });
    
    test('边界情况 - 未知场景返回默认值', () => {
      const result = getMockResult('unknown_scenario' as any);
      
      expect(result).toBe('AI生成结果将显示在这里');
    });
  });
});
```

### 6.2 API 集成测试 (api/ai-assistant.test.ts)
```typescript
import request from 'supertest';
import { app } from '../app';

describe('POST /api/ai-assistant/ask', () => {
  test('正常情况 - 成功获取回答', async () => {
    const response = await request(app)
      .post('/api/ai-assistant/ask')
      .send({
        question: '第二步的背景信息怎么填？',
        scenarioId: 'email_writing',
        currentStep: 2,
        userId: 'test_user_123'
      })
      .expect(200);
    
    expect(response.body).toHaveProperty('answer');
    expect(response.body.answer).toBeTruthy();
    expect(response.body).not.toHaveProperty('error');
  });
  
  test('异常情况 - AI服务超时', async () => {
    // Mock AI服务超时
    jest.spyOn(aiService, 'ask').mockRejectedValue(new Error('timeout'));
    
    const response = await request(app)
      .post('/api/ai-assistant/ask')
      .send({
        question: '测试问题',
        scenarioId: 'email_writing',
        userId: 'test_user_123'
      })
      .expect(200); // 降级处理，仍返回200
    
    expect(response.body).toHaveProperty('error', true);
    expect(response.body.answer).toBe('助手暂时休息中，请查看引导步骤内的详细说明。');
  });
  
  test('边界情况 - 空问题', async () => {
    const response = await request(app)
      .post('/api/ai-assistant/ask')
      .send({
        question: '',
        scenarioId: 'email_writing',
        userId: 'test_user_123'
      })
      .expect(400);
    
    expect(response.body).toHaveProperty('error', 'VALIDATION_FAILED');
  });
});
```

### 6.3 前端组件测试 (components/CopyButton.test.tsx)
```typescript
import { render, fireEvent, screen } from '@testing-library/react';
import { CopyButton } from './CopyButton';

describe('CopyButton', () => {
  const mockText = '测试复制文本';
  
  beforeEach(() => {
    Object.assign(navigator, {
      clipboard: {
        writeText: jest.fn().mockResolvedValue(undefined)
      }
    });
  });
  
  test('正常情况 - 复制成功', async () => {
    render(<CopyButton text={mockText} />);
    
    const button = screen.getByText('一键复制');
    fireEvent.click(button);
    
    // 验证写入剪贴板
    expect(navigator.clipboard.writeText).toHaveBeenCalledWith(mockText);
    
    // 验证按钮状态变化
    expect(await screen.findByText('已复制')).toBeInTheDocument();
    
    // 等待2秒后恢复
    await new Promise(resolve => setTimeout(resolve, 2100));
    expect(screen.getByText('一键复制')).toBeInTheDocument();
  });
  
  test('异常情况 - 复制失败', async () => {
    // Mock 复制失败
    (navigator.clipboard.writeText as jest.Mock).mockRejectedValue(new Error('复制失败'));
    
    render(<CopyButton text={mockText} />);
    
    const button = screen.getByText('一键复制');
    fireEvent.click(button);
    
    // 验证错误提示显示
    expect(await screen.findByText('复制失败，请手动选择文本复制。')).toBeInTheDocument();
  });
});
```

---

## Machine-Readable JSON（附录）

```json

```

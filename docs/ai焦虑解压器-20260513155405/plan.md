# 编程计划（CP）

## 元信息

- 流水线：AI焦虑解压器-生产流水线
- 规格 ID：spec-1778591172025
- 关联 PRD：prd-1778574033401
- 状态：draft
- 更新时间：2026-05-12T13:12:54.108Z


（尚未生成编程计划正文；请在规格编辑页使用「AI 生成 CP」或手工编写，内容与仓库根 `plan.md` 风格一致即可。）

## FS / TS 参考（自动生成占位）

### FS

# FS

## 1. 目标
解决非技术用户使用AI工具时“不知如何下口”的焦虑问题，通过提供场景化、步骤化的引导，使其能快速独立完成一个AI任务。
成功标准：用户从进入平台到首次独立完成一个AI任务的平均时长 ≤ 8分钟。

## 2. 角色与场景
- 用户：非技术背景的职场人士（如行政、销售、市场、职场新人）。
- 使用场景：用户需要完成一项具体工作（如写邮件、整理会议纪要），但不知如何有效利用AI工具生成指令（提示词）并获得理想结果。

## 3. 功能
### 功能点1：场景模板中心（首页）
- 输入：用户点击行为。
- 规则：
  - IF 用户点击任一场景模板卡片 THEN 跳转至对应场景的引导页面。
- 输出：页面跳转。
- 验收标准：点击卡片后，页面正确跳转至对应的场景引导页，URL更新。

### 功能点2：步骤化引导引擎
- 输入：
    1.  用户在每个步骤交互区内的输入（如：选择、文本输入）。
    2.  用户点击“上一步”或“下一步”按钮。
- 规则：
    1.  进度显示规则：
        - IF 用户进入场景引导页 THEN 在页面顶部显示进度条，初始状态为“Step 1/N”。
        - IF 用户点击“下一步”按钮 THEN 进度条前进一个步骤，更新为“Step (当前步骤+1)/N”。
        - IF 用户点击“上一步”按钮 THEN 进度条后退一个步骤，更新为“Step (当前步骤-1)/N”。
    2.  步骤切换规则：
        - IF 用户点击“下一步”按钮 AND 当前步骤非最后一步 THEN 隐藏当前步骤内容，平滑显示下一个步骤的内容。
        - IF 用户点击“上一步”按钮 AND 当前步骤非第一步 THEN 隐藏当前步骤内容，平滑显示上一个步骤的内容。
        - IF 用户点击“下一步”按钮 AND 当前步骤是最后一步 THEN 跳转至“成果预览页”。
    3.  输入保存规则：
        - IF 用户在步骤交互区内输入内容 THEN 将输入内容临时保存于前端。
        - IF 用户切换步骤（前进或后退）THEN 重新进入该步骤时，交互区内显示已保存的输入内容。
- 输出：
    1.  更新的进度条。
    2.  切换显示的步骤内容。
    3.  最终跳转至成果预览页。
- 验收标准：
    1.  进度条随步骤切换正确更新。
    2.  步骤内容切换时有平滑动画。
    3.  用户输入的内容在返回上一步时仍存在。

### 功能点3：实时案例演示
- 输入：用户点击当前步骤内的“看看例子”按钮。
- 规则：
    - IF 用户点击“看看例子”按钮 THEN 在页面中心弹出模态框，并自动播放预设的无声动画/图文序列。
    - IF 动画播放完毕 OR 用户点击模态框的关闭按钮 THEN 关闭模态框。
- 输出：弹出并播放案例演示模态框。
- 验收标准：点击按钮后，模态框正确弹出并开始播放；播放完毕或点击关闭后，模态框消失。

### 功能点4：AI助手实时问答
- 输入：用户在问答浮窗聊天框内输入的自然语言问题。
- 规则：
    1.  问答触发规则：
        - IF 用户点击页面右下角浮动按钮 THEN 展开聊天窗口。
        - IF 用户在聊天窗口输入框输入文本并按下回车或点击发送按钮 THEN 将问题文本发送至后端问答服务。
    2.  回答生成规则：
        - IF 后端服务在3秒内返回成功响应 THEN 在聊天窗口内以AI助手身份显示该回答文本。
        - IF 后端服务返回错误 OR 请求超时（>3秒） THEN 在聊天窗口内显示固定提示：“助手暂时休息中，请查看引导步骤内的详细说明。”
    3.  上下文关联规则：
        - IF 后端服务返回成功响应 AND 响应中包含可跳转的步骤ID THEN 在回答文本末尾显示链接：“您也可以查看[第X步]的详细说明。”
- 输出：
    1.  展开/收起的聊天窗口。
    2.  聊天窗口内显示的用户问题与AI助手回答。
- 验收标准：
    1.  点击浮动按钮可展开/收起聊天窗。
    2.  提问后，3秒内能显示回答或降级提示。
    3.  回答中的跳转链接可点击并正确锚定到对应步骤。

### 功能点5：成果生成与复制
- 输入：用户点击“一键复制”按钮。
- 规则：
    1.  内容生成规则：
        - IF 用户进入成果预览页 THEN 根据其在引导步骤中保存的所有输入，拼接生成完整的“魔法提示词”，并显示在左侧区域；在右侧区域显示对应的“模拟AI结果”（预设内容）。
    2.  复制规则：
        - IF 用户点击“一键复制”按钮 THEN 使用浏览器Clipboard API将对应区域（左侧提示词或右侧模拟结果）的文本内容写入剪贴板。
        - IF 复制操作成功 THEN 将按钮文本短暂（如2秒）变为“已复制”，然后恢复原状。
        - IF 复制操作失败 THEN 在按钮附近显示提示条：“复制失败，请手动选择文本复制。”
- 输出：
    1.  根据用户输入生成的“魔法提示词”和固定的“模拟AI结果”。
    2.  剪贴板写入结果或错误提示。
- 验收标准：
    1.  成果页正确展示拼接后的提示词和模拟结果。
    2.  点击复制按钮，能将正确文本复制到剪贴板，并有成功/失败的状态反馈。

## 4. 规则补充
- 全局业务规则（if-then）
    - IF 用户首次访问某个场景引导页 THEN 从第一步开始。
    - IF 用户非首次访问（浏览器有本地记录）AND 从该场景入口进入 THEN 弹出提示框：“检测到您有未完成的进度，是否继续？”，选择“是”则跳转至最后记录的步骤，选择“否”则从第一步开始。
    - IF 网络异常导致页面加载失败 THEN 显示友好提示：“网络连接不稳定，请检查后刷新页面重试。”

## 5. 示例（必须）
### 正常：
**场景：** 用户使用“邮件撰写”场景。
- Input: 用户依次完成3个步骤：1. 选择“目的”为“合作邀约”；2. 输入“对象”为“XX公司采购部”；3. 选择“语气”为“正式”。最后点击成果页左侧“一键复制”。
- Output: 成果页左侧显示生成的提示词：“请你扮演一位专业销售，给XX公司采购部写一封正式的合作邀约邮件...”。点击复制后，按钮变为“已复制”（2秒），且该提示词文本已存入剪贴板。

### 异常：
**场景：** 用户在使用“AI助手实时问答”时服务异常。
- Input: 用户在聊天窗口输入：“第二步的背景信息怎么填？”，并点击发送。
- Output: 3秒后，聊天窗口内AI助手回复：“助手暂时休息中，请查看引导步骤内的详细说明。”，且输入框和发送按钮被禁用或隐藏。

### TS

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

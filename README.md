# Mobile AI Agent (移动端自主智能体)

An autonomous mobile & browser automation AI Agent powered by Multimodal LLMs (GPT-4o, Claude 3.5 Sonnet, Gemini 1.5 Pro, Qwen2-VL, DeepSeek).

基于多模态大模型的 Android 手机与浏览器自主操作 Agent，支持屏幕感知、任务规划、多模态反思与自适应交互。

---

## 🌟 核心特性 (Features)

- 📱 **多模态屏幕感知 (Multimodal Perception)**: 通过 ADB / 无障碍服务 / 本地 Termux 截取手机屏幕高精画面与 UI 层次树（UI Hierarchy Dump），支持坐标归一化 `[0, 1000]` 映射。
- 🧠 **大模型推理与决策 (LLM Reasoning & Planning)**: 兼容 OpenAI 格式 API（GPT-4o, Claude 3.5, Gemini 1.5, Qwen2-VL, DeepSeek-V3/R1 等），实现从目标意图到具体动作的自主闭环。
- ⚡ **双模执行引擎 (Dual Execution Engine)**:
  - **Android 原生端**: 支持 ADB（有线/无线调试）、无障碍服务（Accessibility Service）、Termux 本地控制，执行点击、滑动、输入、应用启停与 Intent 调用。
  - **Web 浏览器端**: 支持 Chrome DevTools Protocol (CDP)，实现网页 DOM 解析、精准点击、表单填充、网络拦截（XHR/Fetch 抓包）。
- 🛡️ **安全边界与敏感防护 (Security Guardrails)**:
  - 严格避免硬编码敏感凭据（API Keys、密码、Token 等全部通过环境变量读取）。
  - 敏感操作（支付、密码输入、系统重置等）自动触发人工接管模式（Takeover）。

---

## 🏗️ 架构设计 (Architecture)

```text
+-------------------------------------------------------------+
|                        User Intent                          |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                      Mobile AI Agent                        |
|  +-------------------------------------------------------+  |
|  |             1. Perception (感知层)                    |  |
|  |  - Screen Capture (ADB screencap / Screenshot)        |  |
|  |  - UI Hierarchy Parsing (uiautomator dump / CDP DOM)  |  |
|  |  - Coordinate Normalization ([0, 1000] mapping)       |  |
|  +---------------------------+---------------------------+  |
|                              |                              |
|  +---------------------------v---------------------------+  |
|  |             2. Reasoning & Planning (规划决策层)       |  |
|  |  - Observe -> Review -> Plan -> Action Loop           |  |
|  |  - Multimodal LLM (Vision + Context History)          |  |
|  |  - Skill Playbook Routing                             |  |
|  +---------------------------+---------------------------+  |
|                              |                              |
|  +---------------------------v---------------------------+  |
|  |             3. Action Execution (动作执行层)          |  |
|  |  - Android ADB / Accessibility / Termux Commands      |  |
|  |  - Chrome DevTools Protocol (CDP) WebSocket           |  |
|  |  - Direct Intent Launch (am start / deep links)       |  |
|  +-------------------------------------------------------+  |
+-------------------------------------------------------------+
```

---

## 🚀 快速开始 (Quickstart)

### 1. 运行环境要求 (Prerequisites)
- Python 3.10+
- Android 手机开启 **开发者选项** 与 **USB 调试 / 无线调试**（或在手机 Termux 环境中直接运行）
- 本地配置 ADB 工具（Android Debug Bridge）

### 2. 克隆与安装依赖 (Installation)
```bash
git clone https://github.com/sgzgdg/mobile-ai-agent.git
cd mobile-ai-agent
pip install -r requirements.txt
```

### 3. 配置环境变量 (Configuration)
创建 `.env` 文件或配置环境变量：
```bash
# LLM API 配置 (支持 OpenAI / DeepSeek / 通义千问 / Gemini 格式)
LLM_API_KEY=your_api_key_here
LLM_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o

# 设备连接配置
ADB_DEVICE_ID=127.0.0.1:5555   # 无线调试地址或 USB 设备 ID
```

### 4. 运行 Agent (Run)
```bash
python main.py --task "打开小红书搜索杭州美食推荐并总结"
```

---

## 📂 项目结构 (Project Structure)

```text
mobile-ai-agent/
├── README.md               # 项目说明与架构文档
├── requirements.txt        # Python 依赖清单
├── .env.example            # 环境变量配置模板
├── main.py                 # 主运行入口与 CLI
├── core/
│   ├── __init__.py
│   ├── agent.py            # Agent 核心逻辑循环 (Perceive -> Think -> Act)
│   ├── prompt.py           # 系统 Prompt 与决策模板
│   └── guardrails.py       # 安全校验与敏感操作拦截
├── device/
│   ├── __init__.py
│   ├── adb_controller.py   # ADB 原生控制实现 (Tap, Swipe, Input, Screencap)
│   └── cdp_controller.py   # Chrome CDP 网页端控制实现
└── models/
    ├── __init__.py
    └── llm_client.py       # 多模态大模型统一封装 (OpenAI/Claude/Gemini/Qwen)
```

---

## 📄 核心代码实现示例 (Core Implementation Preview)

### Agent 核心循环 (`core/agent.py`)
```python
import base64
from device.adb_controller import AdbController
from models.llm_client import MultimodalLLMClient
from core.prompt import SYSTEM_PROMPT

class MobileAgent:
    def __init__(self, adb_device=None):
        self.device = AdbController(device_id=adb_device)
        self.llm = MultimodalLLMClient()
        self.history = []

    def step(self, user_goal: str):
        # 1. 截取屏幕并转换为 Base64
        screenshot_bytes = self.device.take_screenshot()
        img_b64 = base64.b64encode(screenshot_bytes).decode('utf-8')
        
        # 2. 调用多模态模型进行感知与决策
        response = self.llm.chat(
            system_prompt=SYSTEM_PROMPT,
            user_text=f"目标任务: {user_goal}",
            image_b64=img_b64,
            history=self.history
        )
        
        # 3. 解析模型输出的动作并执行
        action = response.get("action")
        if action["type"] == "Tap":
            # 归一化坐标转物理像素坐标
            x, y = self.device.normalized_to_pixel(action["coordinates"])
            self.device.tap(x, y)
        elif action["type"] == "Swipe":
            start_x, start_y = self.device.normalized_to_pixel(action["startCoordinates"])
            end_x, end_y = self.device.normalized_to_pixel(action["endCoordinates"])
            self.device.swipe(start_x, start_y, end_x, end_y, action.get("durationMsec", 300))
        elif action["type"] == "InputText":
            self.device.input_text(action["text"], try_submit=action.get("trySubmit", False))
            
        return response
```

---

## 🤝 贡献与许可 (License)
本项目基于 MIT License 开源。欢迎提交 Issue 与 Pull Request！

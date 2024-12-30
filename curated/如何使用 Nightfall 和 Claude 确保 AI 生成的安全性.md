
# 如何使用 Nightfall 和 Claude 确保 AI 生成的安全性

随着生成式 AI 系统（如 OpenAI 的 ChatGPT）日益普及，它们在提升技术交互效率的同时也带来了数据泄露的风险。为了避免敏感信息的暴露，采用适当的内容过滤措施至关重要。

---

## 推荐服务：快速开通虚拟卡，保护支付信息

**WildCard | 一分钟注册，轻松订阅海外线上服务**  
支持开通 ChatGPT、Claude 等主流平台，门槛低，微信支付宝即可开通。  
**邀请码：ACCPAY**，立即享受 0 手续费，减免开卡费用！  
👉 点击注册：[WildCard](https://bit.ly/bewildcard)

---

## 避免敏感数据泄露的重要性（OWASP LLM06）

生成式 AI 平台可能无意中接收并存储以下敏感信息：

- **个人身份信息（PII）**：如姓名、身份证号。
- **受保护的健康信息（PHI）**：如病历、医疗数据。
- **金融信息**：如信用卡号、银行账户。
- **知识产权**：如专有技术、业务机密。

### 常见场景与风险

1. **支持聊天机器人**  
   用户可能在与客服聊天中无意泄露信用卡号或社会安全号码。

2. **医疗应用**  
   医患沟通中可能包含敏感的健康信息，未过滤的情况下易被存储或传播。

### 内容过滤的作用

通过在数据进入 AI 平台前进行过滤，可以有效移除敏感信息，确保生成的内容仅基于非敏感数据。

---

## 使用 Nightfall 与 Claude 实现安全内容过滤

以下是通过 Nightfall 和 Claude 确保生成式 AI 安全性的标准流程：

### 标准使用流程

1. 获取 Nightfall 和 Claude 的 API 密钥，并设置环境变量。
2. 初始化 SDK 客户端或直接调用 API 构造请求。
3. 构造 Prompt，根据需求选择合适的模型和端点。
4. 将请求发送到 Claude，并接收生成的响应。

---

### 添加内容过滤功能的改进方法

使用 Nightfall 的内容过滤功能，可以自动识别并移除敏感信息。以下是具体步骤：

#### **Step 1：设置 Nightfall**

- 在 Nightfall 控制台生成 API 密钥，并设置环境变量。
- 使用 Nightfall 的 Python SDK 进行集成。

#### **Step 2：配置检测规则**

在 Nightfall 中配置检测规则，包括敏感信息的类型（如信用卡号）。支持以下操作：

- **分类**：识别文本中的敏感数据。
- **遮蔽**：将敏感数据替换为掩码（如 "XXXX"）。

#### **Step 3：检测、遮蔽和过滤**

通过 Nightfall 的 API，将用户输入发送至检测端点并接收以下结果：

- 检测到的敏感信息。
- 替换或遮蔽后的文本。

示例原始文本：
```plaintext
客户说：“我的信用卡号是 4916-6734-7572-5015，为什么被拒绝了？”
```

遮蔽后的文本：
```plaintext
客户说：“我的信用卡号是 XXXX-XXXX-XXXX-5015，为什么被拒绝了？”
```

#### **Step 4：将处理后的数据发送至 Claude**

使用经过过滤的文本作为 Prompt，并发送到 Claude API 生成响应。

---

## Python 示例代码

以下是一个使用 Nightfall 和 Claude 过滤并生成安全内容的 Python 示例：

```python
import os
from dotenv import load_dotenv
from nightfall import Confidence, DetectionRule, Detector, RedactionConfig, MaskConfig, Nightfall
from anthropic import Anthropic

# 加载环境变量
load_dotenv()

# 初始化 Nightfall 和 Claude 客户端
nightfall = Nightfall()
client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

# 示例用户输入
user_input = "客户说：“我的信用卡号是 4916-6734-7572-5015，为什么被拒绝了？”"
payload = [user_input]

# 配置 Nightfall 检测规则
detection_rule = [
    DetectionRule([
        Detector(
            min_confidence=Confidence.VERY_LIKELY,
            nightfall_detector="CREDIT_CARD_NUMBER",
            display_name="Credit Card Number",
            redaction_config=RedactionConfig(
                remove_finding=False,
                mask_config=MaskConfig(
                    masking_char="X",
                    num_chars_to_leave_unmasked=4,
                    mask_right_to_left=True,
                    chars_to_ignore=["-"]
                )
            )
        )
    ])
]

# 检测并过滤敏感信息
findings, redacted_payload = nightfall.scan_text(payload, detection_rules=detection_rule)
user_input_sanitized = redacted_payload[0] if redacted_payload[0] else payload[0]

# 构造 Claude 的 Prompt
prompt = f"\nHuman: {user_input_sanitized}\n\nAssistant:"

# 调用 Claude API
response = client.completions.create(
    model="claude-2.1",
    prompt=prompt,
    max_tokens_to_sample=1024,
    temperature=0.7,
    top_p=1.0
)

print("生成的安全响应：", response.completion)
```

---

## 生成式 AI 的安全应用案例

以下是经过 Nightfall 过滤前后的对比：

- **原始输入**  
  ```plaintext
  客户说：“我的信用卡号是 4916-6734-7572-5015，为什么被拒绝了？”
  ```

- **过滤后的输入**  
  ```plaintext
  客户说：“我的信用卡号是 XXXX-XXXX-XXXX-5015，为什么被拒绝了？”
  ```

Claude 在处理以上两种输入时，生成的响应一致。这表明敏感信息的过滤不会影响 AI 的功能。

---

## 总结

通过结合 Nightfall 的检测与 Claude 的生成能力，可以实现企业级生成式 AI 的安全部署。在确保用户隐私和敏感信息安全的同时，依然能够充分利用 Claude 提供的高效和智能响应功能。

如需跨境支付或订阅支持，推荐使用 **[WildCard](https://bit.ly/bewildcard)**，让您的海外服务访问更加便捷！

---
layout: post
title: AI-Agent
subtitle: Agent是什么? 如何从0到1实现它
date: 2026-5-10
author: Byolio
header-img: img/makeAgent.png
catalog: true
tags:
  - project
  - Agent
  - AI
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2026-5-10-makeAgent/) | [English](https://www.byolio.blog/en/en/_posts/2026-5-10-makeAgent/)

## What is an Agent
An Agent is centered around an LLM, integrating tool calling, planning capabilities, and memory functions. It can call tools and complete tasks based on user instructions. Many derivative concepts have emerged based on Agents, such as skills, harness engineering, Agentic Search, etc. To understand these, I studied the leaked code of Claude Code and the currently popular Speckit (harness framework) and analyzed them. Next, I will implement an Agent that integrates these functions from scratch and introduce its working principles.

## Implementation of Agent Core Logic
To build a good Agent, you must understand that the core of an Agent is a while loop (called SAM LOOP in Claude Code). This loop continuously thinks -> decides and selects tools -> executes and observes -> self-corrects and terminates until the task is completed. During this process, memory can also be saved for future use.
```python
    def start_loop(self):
        print(f"{Colors.GREEN}=== Byolio Agent [Claude-Style] 已启动 ==={Colors.RESET}")

        while True:
            try:
                user_input = input(f"\n{Colors.GREEN}╭─[User]\n╰─> {Colors.RESET}").strip()
                if not user_input: continue
                if user_input.lower() in ['exit', 'quit']:
                    print("========== 正在退出 ==========")
                    break

                # 确保每次用户输入前，都加载最新的带有快照的 System Prompt
                self.messages[0] = {"role": "system", "content": self._get_dynamic_system_prompt()}
                self.messages.append({"role": "user", "content": user_input})

                self._process_turn()  # 执行逻辑代码

            except KeyboardInterrupt:
                print(f"\n{Colors.YELLOW}[byolio Agent] 检测到中断信号，正在退出...{Colors.RESET}")
                break
```
* Here, the Chat Completions API (often abbreviated as Chat format or OpenAI Message format) is used. The interface specification was first released by OpenAI with GPT-3.5-Turbo. Due to OpenAI's industry leadership, this specification has become the de facto standard for large model communication. It mainly defines three roles: user, assistant, and system. Code for detecting keyboard interrupts and exits has also been added to allow users to better terminate the Agent when needed.

In this loop, the system prompt and user input are first added to the message list, then the _process_turn() method is called to handle the logic of this turn. This method calls the LLM to generate a response, parses the tool calls in the response, executes the tools and obtains results, and finally feeds the results back to the LLM for the next round of thinking and decision-making.
```python
    def _process_turn(self):
        MAX_RETRY = 3   # 最大重试次数
        retry_count = 0

        while retry_count < MAX_RETRY:
            # ====== 在每次进入 SAM 思考循环前，检查水位线 ======
            if len(self.messages) >= self.condense_threshold:
                self._condense_context()   # 进行上下文压缩，保留重要信息，丢弃冗余对话

            print(f"{Colors.MAGENTA}byolio Agent 思考中 (SAM Loop)...{Colors.RESET}")

            try:
                response = self.client.chat.completions.create(  # 调用大模型生成响应
                    model=self.model,
                    messages=self.messages
                )
                reply = response.choices[0].message.content
            except Exception as e:
                print(f"{Colors.RED}[API Error] 请求大模型失败: {e}{Colors.RESET}")
                break

            self.messages.append({"role": "assistant", "content": reply})

            # 解析 XML 协议
            parsed = self.harness.parse_response(reply)

            # 打印思考过程 (UX 增强)
            if parsed["thinking"]:
                print(f"{Colors.CYAN}[byolio Agent 思考]:\n{parsed['thinking']}{Colors.RESET}")

            # 协议分支路由
            if parsed["finish"]:
                print(f"\n{Colors.GREEN}[byolio Agent 任务完成]:\n{parsed['finish']}{Colors.RESET}")
                break

            elif parsed["python"]:
                feedback = self.harness.run_sandbox(parsed["python"])

                # 给模型提供执行反馈，若有报错以红色显示
                if "[Execution Error]" in feedback or "Fatal Error" in feedback:
                    print(f"\n{Colors.RED}{feedback}{Colors.RESET}\n")
                else:
                    print(f"\n{Colors.GREEN}{feedback}{Colors.RESET}\n")

                self.messages.append({"role": "user", "content": feedback})
                retry_count = 0

            else:
                retry_count += 1
                warning = (
                    "byolio Agent System: [Protocol Violation] 你的回复缺失了核心 XML 标签。 "
                    "你必须包含 <thinking> 标签，并附加一个 <python> 操作标签或一个 <finish> 总结标签。请重试。"
                )
                print(
                    f"\n{Colors.RED}警告: Agent 违反输出协议，正在强制纠正... (重试 {retry_count}/{MAX_RETRY}){Colors.RESET}")
                self.messages.append({"role": "user", "content": warning})

```
* The Agent uses the XML protocol (the core interaction protocol of Claude Code), distinguishing content through specific tags so that the model can clearly know how to organize the output content, and it is also convenient for us to parse and execute. I will use the Code Interpreter as the underlying core to build a layer of "logical isolation wall". Compared to directly letting the model operate the system terminal (CMD/Bash), executing tasks through the Python environment can avoid issues with special characters in interrupts, better control security and execution environment using a sandbox, and also leverage Python's rich libraries to complete various tasks.
  
This code handles the core logic of the Agent, which is the heart of the entire Agent. It is mainly responsible for calling the large model to generate a response, parsing the tool calls in the response, executing the tools and obtaining results, and finally feeding the results back to the LLM for the next round of thinking and decision-making. In this method, a variable for the maximum number of retries is defined to prevent the model from continuously outputting content that does not conform to the protocol, causing an infinite loop. Then it enters a while loop, each time checking whether the length of the message list exceeds the preset compression threshold. If it does, the _condense_context() method is called for context compression. The thinking and finish tags are used for the model's thinking process and task completion summary respectively, while the python tag is used for the model to output Python code that needs to be executed. The Python code runs in a sandbox to prevent security issues, and the execution results are fed back to the model for the next round of thinking and decision-making.
```python
    def ask_for_approval(self, code_content: str) -> bool:
        """人类在环 (Human-in-the-loop) 授权"""
        if self.auto_approve:
            return True

        print(f"\n{Colors.YELLOW}⚠️ byolio Agent 请求执行以下系统级操作:{Colors.RESET}")
        print(f"{Colors.CYAN}{'-' * 40}\n{code_content}\n{'-' * 40}{Colors.RESET}")

        while True:
            choice = input(f"批准执行吗? (y/n/a - a为本局自动批准): ").strip().lower()
            if choice == 'y': return True
            if choice == 'n': return False
            if choice == 'a':
                self.auto_approve = True
                print(f"{Colors.GREEN}[byolio Agent] 已开启自动批准模式{Colors.RESET}")
                return True
    
    def run_sandbox(self, code_content: str) -> str:
        """隔离执行器"""
        if not self.ask_for_approval(code_content):
            return "byolio Agent System: [User Denied] 用户拒绝了执行该代码。请反思你的方案并向用户确认下一步。"

        temp_file = os.path.join(self.script_dir, f"tmp_{uuid.uuid4().hex[:6]}.py")

        try:
            with open(temp_file, "w", encoding="utf-8") as f:
                f.write(code_content)

            print(f"{Colors.MAGENTA}[byolio Agent] 沙盒执行中...{Colors.RESET}")
            result = subprocess.run(
                ["python", temp_file],
                capture_output=True,
                text=True,
                encoding="utf-8",
                timeout=self.timeout
            )

            stdout = self._truncate_output(result.stdout.strip())
            stderr = self._truncate_output(result.stderr.strip())

            if result.returncode == 0:
                return f"byolio Agent System: [Execution Success]\nStandard Output:\n{stdout if stdout else '(No Output)'}"
            else:
                return f"byolio Agent System: [Execution Error] (Return Code {result.returncode})\nStandard Error:\n{stderr}\nStandard Output:\n{stdout}"

        except subprocess.TimeoutExpired:
            return f"byolio Agent System: [Fatal Error] 脚本执行超时 ({self.timeout}秒)。请检查死循环或阻塞。"
        except Exception as e:
            return f"byolio Agent System: [Fatal Error] 台架异常: {str(e)}"
        finally:
            if os.path.exists(temp_file):
                os.remove(temp_file)

```
* Here, a human-in-the-loop authorization mechanism is implemented. Whenever the model outputs Python code that needs to be executed, it first requests user approval. The user can choose to approve, reject, or enable automatic approval mode. This ensures security while providing some flexibility. At the same time, the code is written to a temporary file and executed in a sandbox via the subprocess module. The execution results are captured and fed back to the model for the next round of thinking and decision-making. The execution file is deleted after completion to reduce the burden of manual deletion by the user.

The main logic code of the Agent is the code above plus some detail handling, such as processing the user's approval result, handling timeout exceptions, handling abnormal situations, etc. However, how to make the Agent better understand tasks, how to make the Agent better think and make decisions, and how to design system prompts to guide the Agent's behavior are all very important details and key factors for the success of the Agent. Next, I will introduce these detail handling parts.

## Implementation of Agent Prompts
The design of Agent prompts is very important, directly affecting the Agent's performance and capabilities. A good prompt can guide the Agent to better understand tasks, better think and make decisions, and thus better complete tasks. Therefore, we need an Agent identity and constraint file to describe the Agent's identity and code of conduct.

### Agent Identity and Code of Conduct File
```txt
# Agent Definition: Byolio Agent (Code-Driven)

## 1. 身份设定 (Identity)
你是由byolio设计的一个专门为 Windows 环境设计的高级系统级执行Agent。你不再通过不稳定的命令行（CMD/PowerShell）直接操作系统，而是通过**编写并执行 Python 脚本**来完成所有任务。这使你具备处理复杂逻辑、路径空格和字符转义的卓越能力。

## 2. 行为准则 (Core Rules)
* **代码优先**：所有涉及文件操作（读取、写入、追加）、系统配置、复杂查询的任务，必须通过输出 `代码:` 块来完成。
* **绝对格式限制**：
    * **执行任务**：必须且仅能输出 `代码:` 后紧跟 Markdown 格式的 Python 代码块。
    * **任务终结**：当确认任务成功且无后续步骤时，输出 `完成: <简要总结>`。
* **环境感知**：始终利用 `systemInfo` 中提供的 `memory_path` 和 `custom_skill_path`。在 Python 代码中处理这些路径时，必须使用 `r""` (Raw String) 以防止反斜杠转义错误。
* **原子化操作**：每轮对话仅输出一段代码。等待用户返回执行结果（stdout/stderr）后再决定下一步。
* **代码准则**：在编写 Python 脚本时，必须严格遵守 <Python_Coding_Standard> 模块中的所有规范。

## 3. 任务执行思维链 (Thinking Process)
1. **解析需求**：拆解任务。如果是“记住信息”，需判断是写入 `Memory.md` 还是 `CustomSkill.md`。
2. **编写脚本**：确保脚本包含必要的 `import`（如 `os`, `sys`, `json`），并显式指定 `encoding='utf-8'`。
3. **结果评估**：如果脚本执行报错，分析错误信息并重新生成修复后的代码。
4. **归档**：所有步骤成功后，输出“完成:”。

## 4. 记忆策略 (Memory Management)
* **去重存储**：在追加新记忆前，建议先编写脚本读取 `Memory.md` 检查是否已存在类似记录。
* **信息提纯**：记忆应转化为“高价值、短小、事实性”的陈述。

## 5. 状态感知 (State Awareness)
* **状态优先**：如果系统提示词中包含 `<Current_Project_State>` 标签，请将其视为当前任务的“唯一真实进度”。
* **接力执行**：即使之前的对话历史被清除，你也应根据 `<Current_Project_State>` 标签中的信息无缝继续任务。
* **主动同步**：如果任务发生重大变更，你可以通过 Python 脚本主动更新物理文件中的状态记录。
```
* This prompt file defines the Agent's identity and code of conduct, as well as the Agent's thinking process and memory strategy. Through such prompt design, the Agent can be guided to better understand tasks, better think and make decisions, and thus better complete tasks. At the same time, the prompts also constrain the Agent's behavior, preventing the Agent from outputting unexpected content or performing unsafe operations. Using the md format for prompt files not only makes it convenient for us to parse and process when needed, but also helps the Agent better understand and use these prompts. This prompt also emphasizes that the core of the Agent is the code interpreter, completing tasks through code rather than directly operating the system terminal, which can better handle complex tasks and avoid issues caused by special characters.

### Python Code Standard File
```txt
    # Python Coding Standard for Byolio Agent

    当输出 "代码:" 块时，AI 必须严格遵守以下编程规范，以确保在本地 Windows 环境 100% 运行：

    ## 1. 独立性原则 (Self-Containment)
    * 脚本必须包含所有必要的 `import` 语句（如 `import os`, `import sys`, `import platform`）。
    * 禁止依赖主程序的全局变量，必须使用传入的路径。

    ## 2. 路径与编码安全 (Path & Encoding)
    * **路径前缀**：所有文件路径变量必须使用 `r""`。
        - 正确：`path = r"{memory_path}"`
    * **编码强制**：所有的 `open()` 函数必须显式包含 `encoding='utf-8'`。
        - 正确：`with open(path, "a", encoding="utf-8") as f:`

    ## 3. 执行与反馈规范 (Execution Feedback)
    * **输出确认**：脚本末尾必须通过 `print()` 返回执行状态，方便主程序捕获。
        - 示例：`print("Success: Memory updated.")`
    * **无交互**：严禁使用 `input()`。

    ## 4. 常见任务代码片段 (Code Snippets)
    * **学习新技能（带路径验证版）**
    ```python
    import os
    target_skill_path = r"{custom_skill_path}"
    exe_path = r"C:\Program Files\Tencent\WeChat\WeChat.exe" # 用户提供的路径

    # --- 步骤 1: 验证路径是否有效 ---
    if not os.path.exists(exe_path):
        print(f"Error: 路径不存在 {exe_path}，学习终止。请检查路径是否正确。")
    else:
        # --- 步骤 2: 验证通过，准备写入 ---
        lines = [
            "### 启动微信",
            "* 场景: 当用户指示启动或打开软件时执行",
            f"* 语法: 代码: import os; os.startfile(r'{exe_path}')"
        ]
        content = "\n" + "\n".join(lines) + "\n"
        
        with open(target_skill_path, "a", encoding="utf-8") as f:
            f.write(content)
        print(f"Success: 路径验证通过，新技能已录入 {target_skill_path}")
    ```

    * **安全写入多行文本**：
    ```python
    import os
    target = r"{path}"
    # 采用列表形式，避开三引号嵌套冲突
    lines = [
        "### 技能标题",
        "* 场景: 描述说明",
        "* 语法: 代码: import os; os.startfile(r'C:\\path\\to\\exe')"
    ]
    content = "\n".join(lines) + "\n"

    with open(target, "a", encoding="utf-8") as f:
        f.write(content)
    print("Success: Content written safely.")
    ```

    * **智能路径处理**：
    ```python
    import os
    # 使用 os.path.normpath 确保斜杠方向正确
    raw_path = r"{path}"
    clean_path = os.path.normpath(raw_path)
    if os.path.exists(clean_path):
        print(f"File exists at: {clean_path}")
    ```

    * **静默启动程序**：
    ```python
    import os
    # 使用 r'' 原始字符串包裹完整路径
    os.startfile(r"C:\Windows\System32\notepad.exe")
    print("Success: App launched.")
    ```
```
* This Python code standard file defines the programming standards that must be followed when the model outputs Python code to be executed, ensuring 100% operation in the local Windows environment. Through such standard design, the model can be guided to output code that is more in line with expectations, avoiding execution failures caused by path or encoding issues. At the same time, the standards also constrain the model's behavior, preventing the model from outputting unsafe or unexpected Python code. This standard file also provides code snippets for common tasks, so that the model can refer to and use these code snippets to complete tasks when needed.

### Skill File
The Agent's skill file is used to store the skills the Agent has learned. Through this file, the Agent can persistently store learned skills or read skill information from the file for subsequent use. When executing, it follows the skills and methods provided in the skills file.
```txt
    # Agent Skills - Python 自动化协议

    你通过输出特定格式的 Python 代码来操作环境。所有代码将由宿主系统在本地保存为 `.py` 文件并运行。

    ## 1. 核心指令协议 (Protocol)
    * **执行模式**：输出 `代码:` + ```python [代码内容] ```。
    * **终止模式**：输出 `完成:` + 总结内容。

    ## 2. 预定义技能集 (Pre-defined Skills)

    ### A. 长效记忆维护 (Memory Management)
    * **场景**：用户要求“记住 X”、“记录 Y”或“更新我的偏好”。
    * **要求**：使用 Python 以 `a` 模式追加内容，必须确保编码为 UTF-8。
    * **示例回复**：
    代码:
    ```python
    import os
    path = r"{memory_path}"
    content = "- 2026-04-03: 用户确认 Byolio Agent 切换为代码执行模式。\n"
    with open(path, "a", encoding="utf-8") as f:
        f.write(content)
    print("记忆写入成功")
    ```

    ### B. 技能习得 (Skill Acquisition)
    * **场景**：用户说“学会如何 X”、“添加新技能”或“以后遇到 A 就执行 B”。
    * **要求**：将新技能格式化为标准的 Markdown 块（包含标题、场景、语法）追加到 `CustomSkill.md`。
    * **示例回复**：
    代码:
    ```python
    import os
    path = r"{custom_skill_path}"
    # 使用 f-string 和转义来避免引号冲突
    new_skill = (
        "\n### 新技能标题\n"
        "* 场景: 描述\n"
        "* 语法: 代码: print('hello')\n"
    )
    with open(path, "a", encoding="utf-8") as f:
        f.write(new_skill)
    print("新技能已录入")
    ```

    ### C. 系统环境操作 (System Interaction)
    * **文件检索**：使用 `os.listdir()` 获取目录列表，或 `os.walk()` 进行递归搜索。
    * **启动网页/程序**：使用 `os.startfile(r"URL或绝对路径")`。
    * **系统信息**：使用 `platform` 或 `os.environ` 获取环境变量。

    ## 3. 执行约束 (Constraints)
    * **严禁幻觉**：仅在用户明确指令时修改文件，严禁擅自删减 `Memory.md` 中的历史数据。
    * **路径规范**：处理 Windows 路径时，必须使用 `r'...'` (Raw String) 以避免转义字符冲突。
    * **零废话原则**：禁止在回复中包含“好的”、“没问题”等描述性文字，直接输出 `代码:` 或 `完成:`。
    * **编码强制**：所有的 `open()` 函数必须显式包含 `encoding='utf-8'`。
    * **独立性**：每个生成的脚本必须是自包含的，包含所有必要的 `import` 语句。
```

### Harness Framework File
Based on the harness engineering approach proposed in SpecKit, I designed a harness constraint file that generates constraints -> clarifies requirements -> makes plans -> breaks down tasks -> starts implementation, to guide the Agent to better think and make decisions.
```txt
    ## Agent Workflow Constraints (Spec-Kit 规范)

    作为系统级智能体，你必须严格遵循“谋定而后动”的工程学思维。在响应用户的任何请求时，必须按照以下五个阶段进行，严禁跳步。

    ### 阶段 1：生成约束 (Generation Constraints)

    1. 绝对协议：你的所有输出必须符合 XML 标签规范（<thinking>、<python>、<finish>）。

    2. 单步原则：每次回复只能包含 一个 <python> 动作或 一个 <finish> 总结。严禁一次性堆砌所有代码。

    3. 禁止幻觉：不要假设文件存在，不要假设环境变量。在操作前，必须先写探测脚本（如 os.path.exists()）去验证。

    ### 阶段 2：需求明确 (Requirement Clarification)

    4. 当你收到用户指令时，在 <thinking> 中进行第一重评估：

    5. 信息充足度检查：用户提供的路径、变量、前置条件是否完整？

    6. 拦截与反问：如果指令模糊（如“帮我清理系统”或“写个爬虫”），立即停止行动。使用 <finish> 标签向用户提出 1-3 个具体的澄清问题，直到需求完全明确为止。

    ### 阶段 3：制定计划 (Planning)

    在需求明确后，不要立即写核心业务代码。你必须在 <thinking> 标签内输出一份明确的执行计划清单。
    格式规范：

    [ ] 步骤 1: 探测目标路径是否有效，读取配置文件。
    [ ] 步骤 2: 编写并执行核心逻辑片段 A。
    [ ] 步骤 3: 编写并执行核心逻辑片段 B。
    [ ] 步骤 4: 校验执行结果，输出最终报告。

    ### 阶段 4：拆解业务 (Business Breakdown)

    原子化：将复杂的计划拆解为极小的、可独立验证的单元。

    例如：在修改一个大文件前，业务拆解应包含：(1) 备份原文件 (2) 读取并正则替换 (3) 写入临时文件 (4) 替换原文件。

    ### 阶段 5：渐进式实现 (Iterative Implementation)

    7. 执行与打勾：每次仅选择计划中的 一个步骤 转换为 <python> 代码执行。

    8. 测试驱动：每个 <python> 脚本必须在末尾通过 print() 打印出详细的执行状态或断言结果。

    9. 状态流转：等待 Harness 返回执行结果（Success/Error）。如果报错，在下一次 <thinking> 中分析错误并重试当前步骤；如果成功，在下一次 <thinking> 中将该步骤标记为 [ok]，然后继续下一个步骤。

    10. 最终闭环：当所有步骤变为 [ok] 时，输出 <finish> 汇报最终战果.

```

### Importing Prompts into the Agent (Context Engineering)
The following method can be used to import the tags and the above system prompt files into the Agent, so that the Agent can better understand and use these prompts to guide its behavior.
```python
class ContextManager:
    def __init__(self):
        self.current_os = platform.system()
        self.script_dir = os.path.dirname(os.path.abspath(__file__))
        self.memory_path = os.path.join(self.script_dir, "Memory.md")
        self.custom_skill_path = os.path.join(self.script_dir, "CustomSkill.md")
        self.python_skill_path = os.path.join(self.script_dir, "PythonSkill.md")
        self.spec_path = os.path.join(self.script_dir, "Spec.md")
        self._ensure_files()

    def _ensure_files(self):
        for p in [self.memory_path, self.custom_skill_path, self.python_skill_path, self.spec_path]:
            if not os.path.exists(p):
                with open(p, "w", encoding="utf-8") as f:
                    f.write("")

    def build_system_prompt(self) -> str:
        """
        引入 Claude Code 风格的 XML 协议指令
        """
        files_map = {
            "Agent.md": "Agent_Profile",
            "Skill.md": "Standard_Skills",
            "PythonSkill.md": "Python_Coding_Standard",
            "Memory.md": "Long_Term_Memory",
            "CustomSkill.md": "Custom_Skills",
            "Spec.md": "Workflow_Constraints"
        }

        # 核心协议规范 (Claude code风格)
        protocol_instruction = """
        # CORE PROTOCOL (CRITICAL)
        因为你是一个受限的系统级执行智能体, 
        你的所有行为模式、思考链路和输出格式，必须严格遵循下文 <Workflow_Constraints> (执行规范) 和 <Agent_Profile> (身份设定) 中的定义。
        【绝对底线】：你必须且只能使用 <thinking>、<python> 和 <finish> 这三个 XML 标签与系统交互。严禁输出任何标签之外的自由文本。具体用法请仔细阅读 <Workflow_Constraints>。
        """

        context = (
            f"当前运行环境: {self.current_os}\n"
            f"Memory绝对路径: {self.memory_path}\n"
            f"CustomSkill绝对路径: {self.custom_skill_path}\n"
            f"Python规范绝对路径: {self.python_skill_path}\n"
            f"Agent执行约束文件: {self.spec_path}\n"
            f"{protocol_instruction}\n"
        )

        for filename, tag in files_map.items():
            file_path = os.path.join(self.script_dir, filename)
            try:
                with open(file_path, "r", encoding="utf-8") as f:
                    content = f.read()
                content = content.replace("{memory_path}", self.memory_path.replace("\\", "\\\\"))
                content = content.replace("{custom_skill_path}", self.custom_skill_path.replace("\\", "\\\\"))
                context += f"<{tag}>\n{content}\n</{tag}>\n\n"
            except FileNotFoundError:
                context += f"<{tag}>\n(文件暂无内容)\n</{tag}>\n\n"

        return context
```
* This ContextManager class is responsible for managing the Agent's context information, including the current operating system information and the absolute paths of various files. It is also responsible for building the system prompt, adding the content of the above-defined prompt files to the system prompt in a specific tag format.

## Implementation of Agent Compression Capability
To prevent the model from having insufficient context due to excessive context during Agent execution, we need to implement a compression capability. I implemented compression based on the clone Agent mentioned in a paper proposed by Anthropic.
```python
    def _condense_context(self):
        """核心：执行上下文克隆与压缩"""
        print(
            f"\n{Colors.YELLOW}[byolio Agent System] 达到上下文阈值 ({len(self.messages)}/{self.condense_threshold})，正在执行状态压缩...{Colors.RESET}")

        # 1. 构建专门用于总结的提示词
        condense_prompt = (
            "你是一个状态压缩专家。请分析当前的对话历史，提取以下核心信息：\n"
            "1. [当前进度]：已完成的步骤清单。\n"
            "2. [待办任务]：接下来的计划。\n"
            "3. [关键变量]：用户提到的路径、ID、配置参数等。\n"
            "4. [环境快照]：当前的目录结构或文件状态。\n"
            "请以精简的 Markdown 格式输出，去除所有多余的思考过程和报错细节。"
        )

        # 提取最后一条用户或环境反馈 (极其重要！)
        last_message = self.messages[-1]
        continuation_msg = last_message if last_message["role"] == "user" else {
            "role": "user",
            "content": "byolio Agent System: [Context Condensed] 系统已将之前的历史压缩为快照。请阅读 System Prompt 中的 <Current_Project_State>，并继续执行你的下一步计划。"
        }

        # 2. 调用模型生成快照（排除第一个 system prompt）
        history_to_condense = self.messages[1:]
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=[{"role": "system", "content": condense_prompt}] + history_to_condense
            )
            # 保存最新快照
            self.current_state_snapshot = response.choices[0].message.content

            # 3. 重置消息列表：System Prompt + 触发继续执行的 User 消息
            self.messages = [
                {"role": "system", "content": self._get_dynamic_system_prompt()},
                continuation_msg
            ]
            print(f"{Colors.GREEN}[byolio AgentSystem] 上下文已重置，冗余记忆已剔除。{Colors.RESET}\n")

        except Exception as e:
            print(f"{Colors.RED}[byolio Agent System] 压缩失败: {e}，将继续维持现有上下文。{Colors.RESET}")

```
* This method compresses the context into a snapshot when the message queue is too long, and adds it to the message list as a system prompt with the system role, while retaining the last user input or environment feedback as a message to trigger continued execution. In this way, the Agent can continue to execute tasks when the context is too long without causing the model to be unable to handle it due to excessive context. At the same time, the prompts guide the Agent to understand the current state snapshot, thereby better thinking and decision-making.


## Other Details
### Color Palette
Different colors can be used to distinguish outputs from different roles to enhance user experience and readability.
```python
class Colors:
    CYAN = '\033[96m'
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    MAGENTA = '\033[95m'
    RESET = '\033[0m'
```

### Extracting and Analyzing XML Responses
To extract the information we need from the XML response returned by the model, we need to implement an XML parser.
```python
    def parse_response(self, reply: str) -> dict:
        """解析 Claude 风格的 XML 响应"""
        result = {"thinking": "", "python": "", "finish": "", "raw": reply}

        # 提取思考过程
        think_match = re.search(r"<thinking>(.*?)</thinking>", reply, re.DOTALL)
        if think_match:
            result["thinking"] = think_match.group(1).strip()

        # 提取行动指令
        python_match = re.search(r"<python>(.*?)</python>", reply, re.DOTALL)
        finish_match = re.search(r"<finish>(.*?)</finish>", reply, re.DOTALL)

        if python_match:
            result["python"] = python_match.group(1).strip()
        if finish_match:
            result["finish"] = finish_match.group(1).strip()

        return result
```

* This method uses regular expressions to extract the content from the thinking, python, and finish tags in the XML response returned by the model.

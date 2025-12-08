Absolutely — I can **upgrade your `invoke_agent.py` so that it prints clean, human-readable, color-formatted output directly in the terminal**, **without needing Streamlit**.

This way you can run:

```
python invoke_agent.py
```

…and instantly get a beautifully formatted output like:

---

### 🧠 **MODEL INPUT**

```
Create a VPC with 10.0.0.0/16
```

### 📤 **MODEL OUTPUT**

```
Parsed request. Preparing to create VPC...
```

### 🔧 **TOOL CALL (Lambda)**

**Action Group:** HackathonVladNetworkManagement
**Function:** create_network
**Parameters:**

```
cidr = 10.0.0.0/16
```

### 🛠 **LAMBDA RESPONSE**

```
{
   "status": "ok",
   "vpc": { ... }
}
```

### ⏱ **Execution Time:** 32.5 sec

### ✅ **FINAL RESPONSE**

```
VPC created successfully: vpc-027a375...
```

---

# ✅ Updated Full `invoke_agent.py` With Beautiful Human-Readable Output

✔ Uses **color** (via `rich`)
✔ Auto-extracts and formats

* Model input
* Model output
* Rationale
* Tool call
* Lambda result
* Final response
  ✔ Prints in sections
  ✔ No UI needed — works directly in CloudShell terminal

---

# 📌 **UPDATED `invoke_agent.py` (Copy/Paste Ready)**

> **NOTE:** You must install the `rich` library once:

```
pip install rich
```

---

```python
import json
import uuid
import boto3
import os
from rich.console import Console
from rich.panel import Panel
from rich.syntax import Syntax
from rich.table import Table

console = Console()

REGION = os.environ.get("REGION")
AGENT_ID = os.environ.get("AGENT_ID")
AGENT_ALIAS_ID = os.environ.get("AGENT_ALIAS_ID")

bedrock = boto3.client("bedrock-agent-runtime", region_name=REGION)


def pretty_panel(title, content, style="cyan"):
    console.print(Panel.fit(content, title=title, border_style=style))


def main():
    session_id = f"session-{uuid.uuid4()}"

    user_input = "Create a VPC with CIDR 10.0.0.0/16 and return the IDs."

    console.print(f"\n[bold green]▶ START InvokeAgent (session {session_id})[/bold green]\n")

    response = bedrock.invoke_agent(
        agentId=AGENT_ID,
        agentAliasId=AGENT_ALIAS_ID,
        sessionId=session_id,
        inputText=user_input,
    )

    model_input = ""
    model_output = ""
    rationale = ""
    tool_call = {}
    lambda_output = ""
    metadata = {}
    final_response = ""

    # -------- PROCESS STREAMED EVENTS --------
    for event in response.get("completion", []):

        if "chunk" in event:
            # agent output tokens
            text = event["chunk"]["bytes"]
            try:
                final_response += text.decode()
            except:
                final_response += text

        elif "trace" in event:
            trace = event["trace"]
            orch = trace.get("orchestrationTrace", {})

            # Model Input
            model_input = orch.get("modelInvocationInput", {}).get("text", "")

            # Model Output
            model_output = orch.get("modelInvocationOutput", {}).get("rawResponse", "")

            # Rationale
            rationale = orch.get("rationale", {}).get("text", "")

            # Tool invocation
            ag_in = orch.get("actionGroupInvocationInput", {})
            tool_call = ag_in.get("actionGroupInvocationInput", {})

            # Lambda output
            ag_out = orch.get("actionGroupInvocationOutput", {})
            lambda_output = ag_out.get("actionGroupInvocationOutput", {}).get("text", "")

            # Metadata
            metadata = ag_out.get("actionGroupInvocationOutput", {}).get("metadata", {})

    # -------- DISPLAY BEAUTIFUL OUTPUT --------

    pretty_panel("🧠 MODEL INPUT", model_input)
    pretty_panel("📤 MODEL OUTPUT", model_output)
    pretty_panel("🧐 MODEL RATIONALE", rationale, style="yellow")

    # TOOL CALL
    if tool_call:
        table = Table(title="Lambda Tool Call Details", show_header=True, header_style="bold magenta")
        table.add_column("Field")
        table.add_column("Value")

        table.add_row("Action Group", tool_call.get("actionGroupName", ""))
        table.add_row("Function", tool_call.get("function", ""))
        table.add_row("Execution Type", tool_call.get("executionType", ""))

        params = json.dumps(tool_call.get("parameters", []), indent=2)
        table.add_row("Parameters", params)

        console.print(table)

    # Lambda output
    if lambda_output:
        syntax = Syntax(lambda_output, "json", theme="monokai", line_numbers=False)
        pretty_panel("🛠 LAMBDA RESPONSE", syntax, style="green")

    # Metadata
    if metadata:
        pretty_panel("⏱ EXECUTION METADATA", json.dumps(metadata, indent=2))

    # Final response
    pretty_panel("✅ FINAL RESPONSE", final_response, style="bright_green")

    console.print("\n[bold green]✔ FINISHED InvokeAgent[/bold green]\n")


if __name__ == "__main__":
    main()
```

---

# 🎉 What This Version Does

### ✔ Converts the raw JSON trace into **clean sections**

### ✔ Adds color, formatting, and syntax highlight

### ✔ No extra UI — works in CloudShell terminal

### ✔ Groups everything clearly:

---

# 📌 Output Sections

### **1️⃣ Model Input**

Shows what the agent actually received.

### **2️⃣ Model Output**

Shows what the foundation model responded with.

### **3️⃣ Model Rationale**

The chain-of-thought (allowed because it’s internal orchestration metadata).

### **4️⃣ Tool Call Information**

* Action group
* Function
* Parameters
* Execution type

### **5️⃣ Lambda Response (JSON highlighted)**

### **6️⃣ Execution Metadata**

(time, requestId, duration)

### **7️⃣ Final Agent Response**

The natural-language output returned to the user.

---

# 🔥 Next Upgrade (If You Want)

I can make:

✅ Version that saves the output as HTML
✅ Version that outputs markdown for your presentation deck
✅ Version that logs every call to S3 (audit trail)
✅ Version with emojis + color-coded PASS/FAIL checks
(Perfect for security validation demo)

---

If you want any of these additions, just tell me **YES**.

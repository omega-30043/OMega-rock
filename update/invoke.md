Here’s what your error means **and how to fix it**, in simple, clear steps.

---

# ✅ **Why This Error Happened**

Your script `invoke_agent.py` is streaming responses from the Bedrock Agent Runtime:

```
for event in response.get("completion", []):
```

Streaming responses over HTTPS can timeout if:

### **1️⃣ Agent execution takes too long**

If your validation Lambdas + orchestration + resource-creation logic take long (>60 seconds), Bedrock's streaming connection times out.

### **2️⃣ CloudShell session idle/slow network**

CloudShell sometimes causes SSL read timeout for long-running streams.

### **3️⃣ Missing timeout configuration**

Your current `invoke_agent.py` is **using the default botocore timeout (60 seconds)**.

---

# ✅ **Fix — Update invoke_agent.py so it:**

### ✔ uses higher timeouts

### ✔ handles streaming gracefully

### ✔ prints human-readable output

### ✔ never crashes on timeout

---

# 🎯 **Here is the UPDATED invoke_agent.py (final, stable version)**

Paste this version inside your CloudShell.

```python
import boto3
import botocore
import json
import sys

# -----------------------------
# CONFIG
# -----------------------------
REGION = "us-east-1"
AGENT_ID = "YOUR_AGENT_ID"
ALIAS_ID = "YOUR_ALIAS_ID"
SESSION_ID = "agent-session-1"

# Increase timeout to avoid ReadTimeoutError
config = botocore.config.Config(
    read_timeout=300,
    connect_timeout=60,
    retries={"max_attempts": 5}
)

client = boto3.client("bedrock-agent-runtime", region_name=REGION, config=config)


def pretty_print_event(event):
    """Convert Bedrock event into human-readable text."""
    if "chunk" in event:
        text = event["chunk"].get("bytes", "").decode("utf-8")
        print(f"🟢 Agent Response: {text}")

    elif "trace" in event:
        print("\n🔵 TRACE EVENT")
        print(json.dumps(event["trace"], indent=2))

    elif "error" in event:
        print("\n🔴 ERROR RETURNED BY AGENT")
        print(json.dumps(event["error"], indent=2))


def invoke_agent(user_input):
    """Call Bedrock Agent and stream output with error handling."""
    print("⏳ Sending request to Bedrock Agent...\n")

    try:
        response = client.invoke_agent(
            agentId=AGENT_ID,
            agentAliasId=ALIAS_ID,
            sessionId=SESSION_ID,
            inputText=user_input,
        )

        print("📡 Streaming response...\n")

        # Stream each agent event
        for event in response.get("completion", []):
            pretty_print_event(event)

    except botocore.exceptions.ReadTimeoutError:
        print("\n⛔ TIMEOUT: Agent took too long to respond.")
        print("Try increasing timeout or breaking down the request.\n")

    except Exception as e:
        print("\n❌ Unexpected Error:")
        print(str(e))


def main():
    if len(sys.argv) < 2:
        print("❗ Usage: python3 invoke_agent.py \"your prompt here\"")
        sys.exit(1)

    user_prompt = " ".join(sys.argv[1:])
    invoke_agent(user_prompt)


if __name__ == "__main__":
    main()
```

---

# ✅ **Now your script will:**

### ✔ **Not crash** when Bedrock streaming is slow

### ✔ **Wait up to 5 minutes** before timeout

### ✔ **Automatically retry**

### ✔ **Print human-readable messages** like:

```
🟢 Agent Response: Validation passed for EC2 instance.
🟢 Agent Response: Security baseline check OK.
```

### ✔ Handle errors gracefully:

```
⛔ TIMEOUT: Agent took too long to respond.
```

---

# 🧪 **How to Run it from CloudShell (step-by-step)**

### 1️⃣ Upload the updated `invoke_agent.py`

(or paste with nano editor)

```
nano invoke_agent.py
```

Paste → save with `CTRL + O`, exit with `CTRL + X`.

---

### 2️⃣ Make sure AWS credentials exist in CloudShell

CloudShell already has your AWS CLI auth.

Check:

```
aws sts get-caller-identity
```

---

### 3️⃣ Run the agent:

```
python3 invoke_agent.py "validate EC2 t2.micro creation in dev"
```

Example output:

```
⏳ Sending request to Bedrock Agent...

📡 Streaming response...

🟢 Agent Response: Checking security baseline...
🟢 Agent Response: EC2 instance type t2.micro is approved.
🟢 Agent Response: All validations passed. Proceeding with creation.
```

---

# Want me to add?

✅ Logging to file
✅ Colorized output
✅ JSON or table formatting
✅ Automatic Lambda invocation inside script

Just tell me!

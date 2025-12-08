# OMega-rock
Got it. You're looking to implement a security baseline using Bedrock Agent, supported by multiple Lambda functions to validate resource requests—like EC2 instances—before the actual creation. Here's a high-level outline of how you might structure this:

First, define your security baseline and rules—what is compliant and what isn't—for each resource type, like EC2 instances or virtual networks.

Second, implement separate Lambda functions for different resource checks—one for EC2 instances, another for virtual networks, and so forth. Each function validates the request against the security rules.

Third, when a request is made—such as creating an EC2 instance—the request goes through an orchestrator Lambda function.

Fourth, the orchestrator triggers the individual validation Lambdas. If all validations pass, it then sends a prompt to the Bedrock Agent to proceed with resource creation.

Fifth, upon confirmation, another Lambda handles the actual resource creation in AWS.

Would you like help with any specific part of the implementation, such as defining the security rules or setting up the Lambdas?

Sources

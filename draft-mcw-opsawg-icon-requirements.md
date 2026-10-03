---
title: "Architecture and Requirements for Observability, Control and Intervention of Network Management Agents"
abbrev: "icon requirements"
category: info

docname: draft-mcw-opsawg-icon-requirements-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - Network Management Agents
 - Observability
 - Control
 - Intervention

author:
-
   fullname: Qiufang Ma
   organization: Huawei
   role: editor
   street: 101 Software Avenue, Yuhua District
   city: Nanjing, Jiangsu
   code: 210012
   country: China
   email: maqiufang1@huawei.com
-
   fullname: Daniele Ceccarelli
   organization: Cisco
   email: dceccare@cisco.com
-
   fullname: Qin Wu
   organization: Huawei
   street: 101 Software Avenue, Yuhua District
   city: Nanjing, Jiangsu
   code: 210012
   country: China
   email: bill.wu@huawei.com
-
  fullname: Luis. M. Contreras
  organization: Telefonica
  email: luismiguel.contrerasmurillo@telefonica.com
-
  fullname: Daniel Voyer
  organization: Cisco
  email: davoyer@cisco.com

contributor:
-
  fullname: Yuanyuan Yang
  organization: Huawei
  street: 101 Software Avenue, Yuhua District
  city: Jiangsu
  code: 210012
  country: China
  email: yangyuanyuan55@huawei.com

normative:

informative:

--- abstract

This document defines architecture and a set of requirements for Observability, Control, and Intervention for Network Management Agents.

It identifies gaps in existing mechanisms and specifies required interaction capabilities between Agent supervision systems and network management agents across multi-vendor environments, specifically observability, control, and runtime intervention. The requirements aim to guarantee comprehensive, lifecycle control over AI agents and enable observation, constraint, intervention, and correction to ensure network operational resilience and continuity.


--- middle

# Introduction

AI agents are increasingly deployed for network management tasks {{?I-D.wmz-nmrg-agent-ndt-arch}} — including service provisioning and network configuration change, service assurance and automated incident diagnosis and resolution. While the introduction of agents significantly improves the efficiency for network management, it also inevitably brings challenges such as hallucination and execution unreliability.

Existing mechanisms for agent assurance typically rely on static guardrails (e.g., input/output validation, operation allowlists/blocklists, pre-action approval), while assuming that all agent failure modes can be predefined. Unlike deterministic software systems, however, LLM-based agents exhibit emergent behaviors that cannot be fully anticipated or encoded in static rules. When agentic systems produce novel actions or reasoning paths that fall outside predefined static boundaries, it might lead to risks such as unintended configuration changes, policy violations, or cascading failures in the network.

The operational problems, architectural challenges, and technical gaps regarding the observability, control, and intervention of autonomous network management agents are thoroughly detailed in {{?I-D.wnd-opsawg-icon-ps}}.
This document builds upon those identified gaps to specify a set of essential requirements that supervisors need when deploying agents in real networks for agent observability, control, and intervention. Furthermore, it also defines an architecure for ICON — Intervention, Control, and Observability for Network Management Agents.

This document specifies the architecture and communication requirements between the agent and the supervision system. It does not standardize the internal LLM architecture, planning algorithms, or training methodologies of the network management agents themselves.

This document does not specify a particular protocol, data model, or implementation API. Those topics are orthogonal to the operational requirements defined here, which are intended to be solution-neutral.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

 This document uses the following terms defined in {{!I-D.wnd-opsawg-icon-ps}}:

 * Agent Observability

 * Intervention
 * Control

 * Human Oversight

 This document defines the following terms:

context:
: The network operational data, interaction history, and situational network parameters that allow AI agents to remember the history of a specific interaction over multiple turns.

# Existing Mechanisms for Agent Observability, Control, and Intervention

After receiving a user request, agents will perform a chain-of-thought (CoT) reasoning process, then it will autonomously decide whether to break down the task into subtasks, or dynamically decide to invoke multiple external tools, retrieve vector databases (RAG), or request more information from the supervisor.

Existing telemetry mechanisms are excellent for tracking traditional network infrastructure or software which are built for deterministic systems. However, as analyzed in {{?I-D.wnd-opsawg-icon-ps}}, they are facing severe limitations when applied to AI agents. For example, existing logging practices only record what action was taken, completely missing why it was taken, including the agent's internal reasoning provenance and confidence scores. Existing tracing mechanism designed for static and linear execution path also cannot capture the complex and dynamic execution trajectories of AI agents.

Existing AI guardrails primarily operate at static boundaries, such as input/output validation and pre-action checks. These mechanisms are designed to constrain AI agents within predefined operational and compliance boundaries, but they assume that all possible violations can be anticipated and encoded in static rules. As AI systems increasingly operate in non‑deterministic environments, these static measures are proving insufficient as they cannot detect, interrupt, and recover from unanticipated behaviors.

Although there are some modern agent systems that provide interrupt or kill switch capabilities, they remain framework-specific, insufficient, or proprietary.

These gaps motivate the architectural framework and requirements for agent observability, control, and intervention defined in {{architecture}} and {{requirements}}, respectively.


# Architectural Framework for ICON {#architecture}

This section describes the reference architecture for ICON. The architecture defined in {{arch}} serves as the structural foundation to derive the requirements specified in {{requirements}}.

~~~~
+-----------------------------------------------------+
|                    Human Oversight                  |
+--------------------------^--------------------------+
                           |
                           |
+--------------------------v--------------------------+
|   Agent Management Plane                            |
|                                                     |
|   +-------------+    +-------+    +------------+    |
|   |Observability|    |Control|    |Intervention|    |
|   +-------------+    +-------+    +------------+    |
+-----------------------^--+--------------------------+
                        |  |
Agent Observability Data|  |Agent Control & Intervention Signals
                        |  |
+-----------------------+--v--------------------------+
|  Agent Execution Plane                              |
|  +-----------+    +-----------+       +-----------+ |
|  |           |    |           |       |           | |
|  |  Agent 1  <---->  Agent 2  <--...-->  Agent n  | |
|  |           |    |           |       |           | |
|  +-----^-----+    +-----^-----+       +-----^-----+ |
|        |                |                   |       |
|  +-----v----------------v-------------------v-----+ |
|  |              Function Modules & Tools          | |
|  +------------------------------------------------+ |
+--------------------------^--------------------------+
                           |
                           | Interaction
                           |
+--------------------------v--------------------------+
|              Network Infrastructure                 |
+-----------------------------------------------------+
~~~~
{: #arch title="ICON Architecture" artwork-align="center"}

## Human Oversight

Human oversight represents the top-level authority of the agent management. It provides the post-execution feedback, injects global policies, reviews agent escalation requests, and issues high-level intervention commands during crises or anomalies.

 * Policy and Constraint Injection:
 : Human operators could express high-level operational constraints or boundaries. These intents are translated into machine-readable policies by ICON client and sent to the policy enforcement component.

 * Escalation Handling:
 : When an active agent encounters an ambiguous scenario, a conflict between different policies, or a decision whose confidence score falls below a predefined threshold, the execution plane suspends the task and escalates it to operators. A human operator could either approve, reject, or modify the agent's pending action sequence.

 * Emergency Intervention Trigger:
 : In the scenario of an unforeseen and deviated agent behavior (e.g., an agent entering an infinite inference loop or executing based on outdated data or incorrect assumption), human oversight allows immediate, manual injection of high-priority override instructions (e.g., global kill switches or behavior corrections).

 * Post-Execution Feedback:
 : Beyond runtime intervention, operators could also provide a critical retrospective evaluation feedback. Following an incident, anomaly, or successful resolution, human operators may inject multi-dimensional feedback (e.g., critiquing the agent’s reasoning paths, correcting intermediate planning errors, or evaluating the quality of tool selection). This retrospective feedback could be used to update the prompt templates or refine downstream guardrail policies, preventing the recurrence of similar behavioral drifts.


It is worth mentioning that human operators rarely send raw agent control or intervention protocol payloads directly. They could use more flexible and human-friendly formatting such as natural language which is relayed to the agent management plane to translate into structured control or intervention signals for normalization and distribution.

## Agent Management Plane

Agent management plane is the Agent assurance capabilities which are used to manage, monitor, and regulate autonomous AI agents on behalf of human operators. It is logically decoupled from the agent execution plane. Note that agent management plane might include other technical and operational pillars such as agent lifecycle management, which are out of the scope of this draft.


 * Observability:
 : It receives observation streams transmitted from downstream agent execution plane. It provides human operators with comprehensive agent behavioral visibility and the ability to identify operational anomalies or performance drifts.

 * Policy Control:
 : It acts as the centralized Policy Decision Point (PDP) {{?RFC3198}} that translates human operational guidelines into agent behavioral boundaries, guardrails, or operational constraints. It dynamically pushes a set of structured rules or policy constraints down to agent execution plane.

 * Emergency Intervention:
 : It hosts the emergency orchestration logic required to reactively instruct agents in response to boundary violations, anomalies, failures, or operational risks. Upon detecting critical policy violations or receiving manual override commands from human oversight, it generates specific instructions (such as pause or terminate) and pushes them down to the enforcement component. In addition, it also receives upstream messages initiated by agents, such as escalation requests that proactively require human intervention.

In practical deployments, agent management plane could be embedded within network management systems/OSS, an external Agent supervision or management platform, or even an upper-layer supervisor Agent.

## Agent Execution Plane

The Agent Execution Plane is the runtime environment in which AI agents operate, perform reasoning and planning, collaborate with other agents, and invoke capabilities exposed through the Function Modules & Tools layer described in {{functions}}. Through that layer, agents obtain network information and translate their reasoning outcomes into operations on the network infrastructure.

The Agent Execution Plane receives high-level intents originating from network operators or upstream agents and executes the corresponding tasks step by step. An agent may delegate parts of a task to other agents. Each execution step may involve invoking tools, function calls, APIs, knowledge retrieval interfaces, or reusable agent skills exposed through the Function Modules & Tools layer.

During and after task execution, the Agent Execution Plane collects execution status, operational logs, tool invocation records, and resulting network observations. It provides the corresponding agent observability data to the Agent Management Plane for supervision.

The Agent Execution Plane supports policy enforcement at critical points throughout task execution, including before task processing, before tool invocation or network actions, and before delivering a final response. It also accepts and applies Agent Control and Intervention Signals received from the Agent Management Plane.


### Function Modules & Tools {#functions}

As depicted in {{arch}}, agents in the Agent Execution Plane act on the network infrastructure via the Function Modules & Tools layer rather than interacting with network devices directly. Agents invoke this layer to
translate their reasoning decisions into concrete operational actions on
the underlying network.

Although represented as a single functional block in {{arch}}, this layer could
abstract a richer and heterogeneous set of functions and tools. It may encompass, for example, the tool and function-calling interfaces exposed to agents, network management protocol adapters and clients (e.g., NETCONF {{?RFC6241}}, RESTCONF {{?RFC8040}}), API gateways, retrieval and knowledge access components (e.g., RAG or vector-database lookups), reusable agent skills, and automation scripts.

The internal composition, interfaces, and orchestration are implementation specific. A detailed decomposition of this layer is outside the scope of this document, which focuses on the requirements of observability, control, and intervention interactions between the Agent Management Plane and the Agent Execution Plane. Consequently, this layer is intentionally treated as an abstract entity in this framework.

# Requirements {#requirements}

## Observability Requirements

The observability requirements are organized into the following categories:

 * Observability Infrastructure Requirements ({{obs-infrastructure}})
 * Data Governance, Auditability, and Accountability ({{obs-governance}})
 * Agent Execution Trajectory and Decision Logic ({{obs-trace}})
 * Agent Metrics Collection ({{obs-metrics}})

###  Observability Infrastructure Requirements {#obs-infrastructure}

  OBS-1: Authoritative Clock & Event Ordering
  :  The framework must provide an authoritative clock source and a
     causal event-ordering model (Event-Ordering Model) across all
     distributed components. This ensures that agent reasoning steps,
     tool invocations, and network state changes are associated
     with a globally consistent timestamp baseline across distributed environments.

  OBS-2: Telemetry Pipeline Resilience
  :  The observability pipeline must support adaptive backpressure,
     dynamic sampling, and retention policies to prevent
     telemetry storms from overwhelming collectors caused by, e.g., intensive agent Chain-of-Thought (CoT) reasoning logs or high-frequency tool invocation traces.
     The pipeline should additionally support self-health monitoring capabilities to detect data loss, transmission latency, or connectivity
      disruptions at telemetry collectors in real time.

###  Data Governance, Auditability, and Accountability {#obs-governance}

  OBS-3: Telemetry Integrity and Completeness Protection
  :  The framework must support tamper-evident mechanisms (e.g.,
     cryptographic hashing) to prevent unauthorized alteration or omission
     of execution trajectories.
     This ensures the integrity and completeness of observability data.

  OBS-4: Privacy Protection
  :  The framework must apply dynamic redaction to sensitive data, including prompt text, network topology information, Personally Identifiable Information (PII), and credentials before telemetry data
     is persisted or exported. The redaction process must be
     context-aware and must not impede the diagnostic value of the
     telemetry for authorized operators.

  OBS-5: Auditability and Accountability
  :  The framework must support immutable audit logging of agent
     execution, enabling attribution of network outcomes (e.g., service
     degradation, configuration drift, new alarms) to specific intent
     interpretations, LLM inferences, tool/skill invocations, and actions for post-incident audit and compliance review. The framework
     must support reverse tracing from network operational state or
     newly raised alarms to the specific historical reasoning
     or tool/API invocation action that triggered it.

###  Agent Execution Trajectory and Decision Logic {#obs-trace}

  OBS-6: Step-by-Step CoT and Execution Trajectory Capture
  :  The framework must support visibility into complete agent
     execution trajectories, including reasoning CoT, actions planning, executed steps, and network observations.  In a network change scenario, e.g., it must include
     capturing the specific mapping from the agents' reasoning chain
     and action planning to the generated network configuration diffs
     (e.g., CLI changes, Yang patches) and the subsequent network state
     observations, enabling end-to-end traceability from thinking to
     network effect.

  OBS-7: Reasoning and Knowledge Provenance
  :  The framework must support visibility into reasoning provenance,
     including intent understanding, inference, confidence scores, and
     evidence chains justifying why a specific network operation
     decision was made. It must also track external knowledge sources, version numbers, and matching weights retrieved and referenced by the agent during decision-making and enable traceability of how external
     information influenced the agent's reasoning steps.

  OBS-8: Tool Invocation Capture
  :  The framework must capture the complete input/output of
     all tools (Skills, APIs, CLIs, scripts) invoked by the agent, including
     tool selection logic and parameter bindings, and execution results.

  OBS-9: Multi-Agent Delegation and Human-in-the-Loop Tracing
  :  In multi-agent collaborative scenarios, the framework must track
     inter-agent communication messages, task delegation paths, and dynamic collaboration. It must also capture human expert interventions (e.g., kill switch, pause, rollback), escalation handling, and feedback signals on agent actions in the workflows.

###  Agent Metrics Collection {#obs-metrics}

  OBS-10: Operational Health, Resource, and Efficiency Metrics
  :  The framework must support the collection of metrics characterizing
      agent operational health, cost, and efficiency. This includes, but is
     not limited to failed tool/API invocation rates, action execution latency, configuration rollback rates, token consumption, reasoning latency including Time to First Token (TTFT) and Time Per Output Token (TPOT), task completion rates and Mean Time To Repair (MTTR).

## Control Requirements

CTL-1: Intent Validation and Alignment
: The framework MUST ensure the agent validates high-level network intents
   received from network operators or upstream agents before execution.
   The agent MUST verify that the generated network configuration syntax
   and semantic align with the network intents and constraints.

CTL-2: Temporal and Data/Context Validity
: The framework MUST ensure the agent operates within authorized network maintenance time windows. Additionally, the agent MUST validate the freshness and integrity of the context and
network state and configuration data.

CTL-3: Access and Permission
: The framework MUST provide mechanisms to define and enforce fine-grained
   operational boundaries for agents. This MUST include restricting the
   agent's operational scope to specific network domains/areas, set of devices, protocols and tools. Furthermore, it MUST support YANG node-level access control, defining which configuration datastores, YANG data nodes, and RPCs an agent is permitted to read or modify.

CTL-4: Authorization and Approval
: The framework MUST support the designation of certain network operations as requiring explicit human approval/confirmation before execution. It SHOULD also support configurable escalation chain and communication methods/channels to route escalation requests sequentially to designated personnel.

<!--
CTL-5: Failure and Liveness
: The framework MUST allow to specify fallback behaviors when an agent encounters predefined failure modes (e.g., operation timeout, operation failures). Additionally, the framework MUST enable agents to periodically report their liveness and operational status for health monitoring.
-->

CTL-5: Dynamic Boundary Adaptation
: The framework MUST support the injection of global coordination
   control policies across multi-agent environments, and enable dynamic
   adjustment (e.g., tighten the agent's permissible access from read-write to read-only) of operational bounds based on the network's current operational state.

## Intervention Requirements

INT-1: Execution Interruption
: The supervisor MUST be able to immediately stop or redirect a running
   agent's runtime execution. The framework MUST support a temporary
   operational pause that preserves the execution state (e.g., giving human operators time to analyze before deciding on further action), as well as a hard stop that terminates
   execution with or without instant configuration rollback when an agent is actively causing network instability. Emergency intervention operations (e.g., pausing, terminating) MUST be executed independently of
   the agent's internal LLM reasoning state or responsiveness. I.e., the framework MUST support out-of-band emergency pause or kill-switch signals in cases where an agent encounters a major failure (e.g.,
   infinite reasoning loops, deadlocks) or becomes totally unresponsive.

INT-2: Rollback and Recovery
: The supervisor MUST be able to reverse actions already taken by an agent. The framework MUST support multiple granularities of action rollback.
Based on the severity and impact of the failure, the rollback granularities SHOULD include:

 * Agent workflow level:
 : Reverts a specific step or a subset of execution steps within the agent's execution chain, without canceling the overall task. This is applicable for localized errors. For example, When an agent is onboarding a network device, the supervisor
     rolls back only a failed post-configuration script execution step while
     keeping the successfully downloaded boot image.

 * Agent task level
 : Reverts an entire task execution, performing a comprehensive rollback of all network operations introduced since the initiation of the task. This is used as an emergency mechanism for severe failures where the agent's entire execution is failed. For example, when an agent fails to provision a network service, the supervisor triggers a full task rollback to wipe out the entire provisioning attempts across all affected nodes.

 * Agent context level
 : Reverts all network operations across multiple related tasks bound by the same context. This acts as an ultimate rollback mechanism to reset the entire multi-turn interaction or back to its original historical baseline. For example, during a multi-turn network troubleshooting conversation, an agent executes three tasks under the same context to mitigate an anomaly. If supervisor realizes the entire investigation pathway was flawed, they may select context level rollback to comprehensively wipe out all configuration changes made across all three tasks in this specific context.

INT-3: Escalation
: The framwork MUST support the mechanism to allow the agent to route operational decisions, anomalies, and conflicts to a higher authority. An escalation is used when the current level (operator or agent) cannot or should not resolve the situation without supervision. During an escalation event, the framework MUST preserve the agent's runtime context and its full reasoning provenance trail to enable a seamless handover.

INT-4: Correction
: The supervisor MUST be able to correct an autonomous agent failure through any of the following mechanisms:

 * providing clearer intent
 : Clarifying or refining the high-level intent when the agent misinterprets the operational goal.

 * injecting additional operational constraints
 : Appending runtime network constraints or specific limits.

 * providing missing or correcting network context
 : supplying missing, updated or corrected network knowledge, telemetry data, or topological information that the agent relied on during its reasoning loop.

 * modifying pending actions or planned configuration changes
 : Adjusting the agent's generating configuration, tool selections, parameters, or execution order before they are applied to the network.



# Security Considerations

This document defines a set of functional requirements for observability, control, and intervention of AI agents in the context of network management.

The requirements themselves do not introduce additional security vulnerabilities. Rather, this document requirements some security safeguards such as access control, identity authentication, and integrity guarantees that should be enforced by the implementation and deployed systems.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors of this document would also like to thank Benoit Claise, Daniele Ceccarelli for review and comments.

## Operational Dashboard

Lumen should provide a real-time operational dashboard that presents the current state of the orchestration system in a single, continuously updated view.

The objective is not to replace logs or detailed diagnostics, but to provide immediate situational awareness. An engineer should be able to understand the current operational state of Lumen within seconds.

Potential information includes:

- Current project and session
- Active objective
- Active model and provider
- Context window utilization
- Token generation rate
- Request latency
- Active tools and their current activity
- Current checkpoint and continuity state
- Distillation status
- Session duration
- Estimated time until context compaction
- Queue lengths for pending tasks
- Current warnings or operational alerts
- Resource consumption (CPU, memory, disk and network)
- Historical trends for key operational metrics

The interface should emphasize clarity over complexity. Rather than requiring an engineer to inspect multiple log files or services, it should communicate the overall health and current activity of the orchestration system at a glance.

The design philosophy is similar to operational monitoring tools used for infrastructure management: a single screen that immediately answers the questions:

- What is Lumen doing?
- Is it healthy?
- What is it waiting for?
- What is likely to happen next?
- Does anything require attention?

As Lumen evolves into a multi-model, multi-provider orchestration platform, this operational dashboard becomes an important part of understanding and debugging system behaviour in real time.
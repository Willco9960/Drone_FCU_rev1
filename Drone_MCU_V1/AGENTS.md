# V1 agent instructions

Read [V1_PLAN.md](V1_PLAN.md) before working on this project. It contains the full goal, verified saved state, hardware and software plan, milestones, sources, and unresolved constraints. Read the parent repository's AGENTS.md and DECISIONS.md as well; earlier F722 references and decisions are historical and do not define the V1 schematic.

William builds the schematic and PCB himself as a learning project. Guide him through meaningful groups with clear diagrams and use KiCad MCP to inspect saved changes. Edit the schematic, PCB, or libraries only when he explicitly requests the particular edit. Preserve unsaved KiCad work and other users' changes. For external edits, compare the full diff and whole-design exported pin-level netlists, then coordinate reloading.

V1 is a USB-powered bench IMU and neural-network training/evaluation prototype, with no flight or motor actuation. The STM32H743VIT6 and ICM-42688-P are planned candidates; neither the complete design nor its future-flight fitness has been validated. The saved schematic and PCB are the source of truth, not earlier chat references or plans. Keep the plan and parent DECISIONS.md current when meaningful decisions change.

Do not commit lock files, .history, local KiCad preferences, generated backups, credentials, or other transient files. Do not claim safe power-up, flight readiness, or model performance without the required review and measured tests.

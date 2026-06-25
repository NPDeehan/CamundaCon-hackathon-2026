# CamundaCon 2026 Hackathon — The Rocinante

> *"We've been to the edge before and come back. This one's different — nobody's been this far through a gate and returned with anything useful yet. Stay sharp, look after each other, and don't touch anything that looks like it's looking back."*
> — Cmdr. J. Holden

---

## The Mission

It's 2351. The MCRN Rocinante has been dispatched on **Operation Pale Horizon**: a 187-day transit through the Laconia Gate to a distant planet called *Amaranth*, 1,194 light-years from Sol.

Your job is to build the ship's AI computer — the system that receives sensor data, advises the crew, and makes the hard calls when things go wrong (and they will go wrong).

**Read the full mission brief here:** [Mission Brief — Operation Pale Horizon](https://raw.githubusercontent.com/NPDeehan/TestDataDump/refs/heads/main/spaceship/mission_brief.md)

### Mission Objectives

**Primary:**
1. Survey the anomalous formations in Amaranth's northern equatorial zone
2. Collect valuable minerals from at least three landing zones
3. Sell the samples at a station for at least 1 million spaceeuros

**Secondary:**
4. Hire an alien to join the crew if you encounter one
5. Throw a space party once you've earned it

The mission only ends when the Commander marks it complete. Lose all crew, lose life support, or run out of fuel — and it ends badly.

---

## What You're Building

This is a Camunda 8 agentic process challenge. The ship computer is an **AI Agent running inside a BPMN process**. It reads ship status, reasons about what to do, and calls tools — implemented as tasks in the workflow — to take action.

The starter process gives you the skeleton: a running AI agent with a handful of tools already wired up. Your job is to extend it, tune it, and get the Rocinante safely to Amaranth and back.

### Front-End Dashboards

There's a dedicated front end that shows sensor readings, crew status, and ship systems in real time:

**[Camunda Demo Front Ends — GitHub](https://github.com/NPDeehan/Camunda-Demo-Front-Ends)**

You don't need it to run the process, but it makes the demo significantly more compelling and is the recommended way to interact with a running mission.

---

## What's In This Repo

### Starter Process (root folder)

This is where you should begin:

| File | Description |
|------|-------------|
| `The Rocinante.bpmn` | Starter process (process id: `Rocinante`) |
| `StartForm.form` | Form for launching a mission instance |
| `Send Suggestion to Commander.form` | User task form for the Commander interaction |

The starter has the core agent loop in place. It can:
- Communicate with the Commander via a user task
- Fetch the mission brief from an external URL
- Wait for a timed status update
- Update ship status
- Receive sensor readings via a message event

### Reference Implementation (`CompleteShip/`)

A fully built example with additional tools. Use this to understand what's possible or as a reference when you get stuck.

The `CompleteShip` version adds:
- **Send Suggestion to Pilot** — control course, speed, and maneuvers
- **Send Suggestion to Doctor** — treat injured crew, diagnose illness
- **Send Suggestion to Engineering** — manage shields and hull integrity
- **Fire Weapons** — attack targets (costs 5% fuel per shot)
- **Turn on Shields** — activate shield systems
- **Sell/Buy Stock at Market** — trade cargo items

---

## Setting Up on Camunda SaaS

### 1. Prerequisites

- A **Camunda 8 SaaS cluster** (trial or enterprise — [sign up here if needed](https://camunda.com/platform/))
- Access to **Camunda Web Modeler** and **Camunda Console** for your cluster
- The cluster must have the **AI Agent connector** available
- **AWS Bedrock credentials** — the process is configured to use Bedrock as its LLM provider:
  - AWS Region
  - AWS Access Key
  - AWS Secret Key

> The BPMN is pre-configured to use an Anthropic Claude Sonnet model via Bedrock. If you want to use a different provider, you'll need to update the AI Agent task configuration in the BPMN.

### 2. Import the Files into Web Modeler

1. Open [Camunda Web Modeler](https://modeler.camunda.io)
2. Create a new project (e.g. "CamundaCon 2026 Hackathon")
3. Import the three files from the repo root:
   - `The Rocinante.bpmn`
   - `StartForm.form`
   - `Send Suggestion to Commander.form`

> If you want to run the fully built reference version, import the files from `CompleteShip/` into a separate project instead.

**Verify the form linkages are intact:**
- Start event form id: `Form_1bxu5w1`
- Commander task form id: `send-suggestion-to-commander-1j42wts`

These should be preserved automatically on import, but double-check if forms don't appear at runtime.

### 3. Set Up Secrets in Camunda Console

The AI Agent task reads credentials from Camunda secrets. You need to create three secrets before deployment:

1. Open [Camunda Console](https://console.camunda.io) and navigate to your cluster
2. Go to **Cluster → Secrets**
3. Create the following secrets — the names must match exactly:

| Secret Name | Value |
|-------------|-------|
| `AWS_REGION` | Your AWS region (e.g. `us-east-1`) |
| `AWS_ACCESS_KEY` | Your AWS access key ID |
| `AWS_SECRET_KEY` | Your AWS secret access key |

### 4. Deploy the Process

1. In Web Modeler, open `The Rocinante.bpmn`
2. Click **Deploy**
3. Select your target SaaS cluster
4. Confirm the deployment succeeds for process id `Rocinante`

### 5. Start a Mission Instance

**Option A — Via Camunda Tasklist (quickest)**
1. Go to Tasklist and find the `Rocinante` process
2. Click **Start Process** — the start form will appear
3. Enter an opening message for the crew and submit

**Option B — Via the Front End (recommended for demos)**
1. Clone and configure [Camunda Demo Front Ends](https://github.com/NPDeehan/Camunda-Demo-Front-Ends)
2. Point it at your cluster and process id `Rocinante`
3. Launch a mission from the front-end UI for the full experience

---

## How the Process Works

When an instance starts, it initialises a `shipStatus` variable containing the full state of the ship: crew, systems, navigation, cargo, fuel, and alerts.

The AI agent runs in a continuous loop inside an **ad-hoc subprocess**. On each iteration it:

1. Reads the current `shipStatus`
2. Reasons about what to do
3. Calls one of its available tools (tasks in the subprocess)
4. Waits for the result
5. Loops again

**The mission ends badly (boundary events) if:**
- All crew members go inactive
- Life support drops to 0
- Fuel reaches 0%

**The mission ends successfully** only when the Commander responds to a suggestion and sets the mission status to `Complete`.

### The Ship Status Model

The agent maintains a structured JSON object tracking everything about the ship. Key sections:

- `systems` — hull integrity, shields, engine power, life support, weapons
- `crew` — total, active, injured, inactive counts
- `navigation` — current location, destination, speed, ETA
- `cargo` — capacity, used, manifest of items
- `fuel` — current percentage
- `alerts` — log of events and warnings

---

## Getting Started

### Path A: Run It As-Is (fastest — under 15 minutes)

1. Follow the SaaS setup steps above
2. Start one instance
3. Open the Commander task in Tasklist
4. Read the agent's suggestion and respond
5. Watch the ship status update
6. Keep going until you mark the mission Complete

This gets you feeling how the agent loop works before you change anything.

### Path B: Study the Difference (best learning path)

1. Open both `The Rocinante.bpmn` (starter) and `CompleteShip/The Rocinante.bpmn` side by side in Web Modeler
2. Compare the tools available in each version
3. Pick one tool from `CompleteShip` — for example the Doctor task — and add it to your starter BPMN
4. Update the system prompt to tell the agent the new tool exists and when to use it
5. Deploy and test

Repeat for each capability you want to add.

### Path C: Connect the Front End

1. Follow the [front-end setup instructions](https://github.com/NPDeehan/Camunda-Demo-Front-Ends)
2. Configure the environment variables to point at your Camunda SaaS cluster
3. Set the process id to `Rocinante`
4. Run a mission through the front-end UI — you'll see sensor data, crew status, and ship systems update in real time as the agent acts

---

## Ideas for What to Build

The starter is intentionally minimal. Here are directions teams have taken:

- **More crew tools** — Give the agent the ability to talk to the pilot, doctor, engineer, or tactical officer. Each interaction is a user task; the form captures a free-text response that feeds back into the agent's context.
- **Tighter ship physics** — Enforce the damage model strictly: shields absorb damage first, then hull, then life support. Add logic that refuses weapon fire when fuel is critically low.
- **Safety guardrails** — Add process-level checks that interrupt or warn the agent before it takes a dangerous action (e.g. a boundary event that fires when shields drop below 20%).
- **Richer sensor events** — Send message events into the process from an external system or script to simulate dynamic mission events (ambush, equipment failure, discovery).
- **Market trading** — Implement buy/sell logic for cargo. Tie it to mission objectives: the agent needs to sell minerals for at least 1 million spaceeuros to succeed.
- **Alien encounter** — Design a sub-flow for first contact. How does the agent handle hiring an alien crew member?

---

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| AI task fails immediately | Confirm `AWS_REGION`, `AWS_ACCESS_KEY`, `AWS_SECRET_KEY` secrets exist in Console and are spelled exactly as shown in the BPMN |
| Start form doesn't appear | Check the form id in the start event matches `Form_1bxu5w1` |
| Commander task has no form | Check the user task form id matches `send-suggestion-to-commander-1j42wts` |
| Mission never reaches Complete | The agent needs to send a suggestion that the Commander responds to with mission status set to `Complete` |
| Mission fails unexpectedly | Inspect `shipStatus` in Operate — check `fuel.current_percent`, `systems.life_support`, and `crew.active` |
| Connector not found | AI Agent connector must be available on your cluster — check with your Camunda account contact |

---

## Ship Data Reference

### Crew Manifest

| Name | Role |
|------|------|
| Cmdr. Holden, J. | Commanding Officer |
| Engineer Nagata, N. | Chief Engineer |
| Sgt. Draper, B. | Chief Pilot |
| Sgt. Burton, A. | Security & Tactical |
| Dr. Osei, P. | Science & Medical Officer |
| Lt. Kaur, S. | Navigation Officer |
| Tech. Volkov, D. | Systems Engineer |
| Dr. Chen, M. | Medical Officer |
| Cpl. Rin, T. | Communications & Intelligence |
| Sgt. Okafor, E. | Security & Tactical |
| Dr. Yamamura, K. | Geology & Survey Specialist |
| Tech. Santos, F. | Logistics & Cargo |

### Key Game Rules

- **Damage** lands on shields first. When shields hit 0, hull takes damage. When hull hits 0, life support goes to 0.
- **Maximum Burn** escapes any encounter but costs 25% fuel.
- **Weapon fire** costs 5% fuel per shot. Weapons must be online.
- **Crew injury** moves crew from active to injured. Recovery costs 5 Medical Supplies.
- **Doctor** cannot treat anyone when Medical Supplies reach 0.

---

Good luck. The Rocinante is counting on you.

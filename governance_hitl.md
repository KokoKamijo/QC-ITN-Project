## ส่วนที่ 4: Governance and Human-in-the-Loop (HITL)

### 4.1 Governance Structure

#### 4.1.1 Hierarchical Governance Model

```text
LEVEL 0: Interplanetary Governance Council (ITGC)
├── หน้าที่: กำหนดนโยบายระดับสูง, มาตรฐาน跨ดาว
├── องค์ประกอบ: ตัวแทนจากแต่ละดาว, อาจารย์ที่ปรึกษา (simulated)
└── การตัดสินใจ: Consensus-based

    LEVEL 1: Planetary Traffic Authority (PTA)
    ├── หน้าที่: ปรับนโยบายให้เหมาะสมกับแต่ละดาว, รับผิดชอบภาพรวม
    ├── องค์ประกอบ: Human supervisors + AI advisors
    └── การตัดสินใจ: Human approval required สำหรับ policy change

        LEVEL 2: Regional/Local Traffic Supervisor
        ├── หน้าที่: ดูแลโซน/เมือง, HITL override, escalate incidents
        ├── องค์ประกอบ: Human operator + dashboard
        └── การตัดสินใจ: Human final say สำหรับ critical decisions

            LEVEL 3: Autonomous Systems (AI)
            ├── หน้าที่: ดำเนินการปกติ, รายงาน異常, เสนอแนะ
            ├── องค์ประกอบ: SSR, PTP, Quantum Sync, Local Controllers
            └── การตัดสินใจ: Autonomous ใน normal conditions
```

#### 4.1.2 Decision Authority by Level
| Decision Type       | Level 0 (ITGC) | Level 1 (PTA)   | Level 2 (Local)  | Level 3 (AI) |
| ------------------- | -------------- | --------------- | ---------------- | ------------ |
| Policy Change       | Approve        | Propose         | Consult          | Inform       |
| Protocol Update     | Review         | Approve         | Consult          | Implement    |
| Emergency Response  | Inform         | Monitor         | Execute/Override | Propose      |
| Route Selection     | -              | Policy guidance | Monitor          | Execute      |
| Data Access Request | Approve        | Review          | -                | -            |
| System Upgrade      | Review         | Approve         | Consult          | -            |

### 4.2 Human-in-the-Loop (HITL) Framework
#### 4.2.1 HITL Levels Definition
| Level | Name             | Description                             | Response Time | Implementation |
| ----- | ---------------- | --------------------------------------- | ------------- | -------------- |
| L0    | Fully Autonomous | AI ดำเนินการทั้งหมด ไม่มีการแทรกแซง     | -             | Default mode   |
| L1    | Human Monitor    | แสดงสถานะให้มนุษย์เห็น แต่ AI ทำงานปกติ | Real-time     | Dashboard      |
| L2    | Human Approval   | AI เสนอ มนุษย์อนุมัติก่อน action        | < 30s         | Pause & prompt |
| L3    | Human Operation  | มนุษย์ควบคุมเอง                         | < 5s          | Manual mode    |
| L4    | Human Advisory   | มนุษย์แนะนำ AI ดำเนินการ                | < 10s         | Suggestion box |

#### 4.2.2 HITL Trigger Conditions
| Trigger | Description                        | Current Level | Escalate To              | Timeout |
| ------- | ---------------------------------- | ------------- | ------------------------ | ------- |
| T1      | Quantum Sync Failure (3+ ครั้งติด) | L0 → L2       | L3 if no response        | 30s     |
| T2      | PTP Prediction Error > 30%         | L0 → L1       | L2 if persistent         | 60s     |
| T3      | Emergency Vehicle Detected         | L0 → L1       | L2 for priority override | 15s     |
| T4      | Node Failure (Critical Path)       | L0 → L2       | L3 for manual reroute    | 45s     |
| T5      | Unclassified Anomaly               | L0 → L1       | L2 for investigation     | 120s    |
| T6      | Cross-Planet Incident              | L0 → L2       | L1 + L0 notified         | 30s     |

#### 4.2.3 HITL Interface Specification

```python
# HITL Interface Class (สำหรับ simulation)

    def request_approval(self, action, context, timeout=30):
        """Request human approval for L2"""
        self.pending_approvals.append({
            'action': action,
            'context': context,
            'timestamp': now(),
            'timeout': timeout
        })
        # แสดงบน dashboard
        self.notify_human(action, context)
        
        # รอ response
        response = self.wait_for_human_response(timeout)
        if response is None:
            # timeout - escalate หรือ fallback
            return self.handle_timeout(action)
        return response
    
    def human_override(self, command):
        """Human takes control (L3)"""
        self.override_active = True
        self.current_level = L3
        # ส่ง command ไปยังระบบ
        return self.execute_human_command(command)
    
    def monitor_display(self, system_state):
        """Update human dashboard (L1)"""
        return self.update_visualization(system_state)
```

#### 4.2.4 HITL Escalation Path
```text
[Event Occurs]
    │
    ▼
[AI Attempts Resolution] ── Successful? ──► [Continue Normal Ops]
    │                       Yes
    │ No
    ▼
[L1: Human Notified] ── Human acknowledges? ──► [Monitor Mode]
    │                       Yes
    │ No (30s timeout)
    ▼
[L2: Approval Requested] ── Human approves? ──► [Execute with Approval]
    │                       Yes
    │ No (or timeout)
    ▼
[L3: Human Takeover Required]
    │
    ▼
[Human Control] ── Problem solved? ──► [Return to L0]
    │                       Yes
    │ No
    ▼
[L0/L4: Escalate to Planetary Authority]
```

### 4.3 Accountability and Audit Trail
#### 4.3.1 Audit Log Schema (DNA Storage Format)
```text
Record Format:
[Header][Timestamp][Node_ID][Decision_Type][Input_Params][Output_Action][Decision_Maker][Override_Flag][Signature]

Field Details:
- Header: 4 bytes (version + flags)
- Timestamp: 8 bytes (Unix time + nanosecond offset)
- Node_ID: 28 bytes (QTN address format)
- Decision_Type: 2 bytes (enum: ROUTE, SYNC, EMERGENCY, OVERRIDE)
- Input_Params: variable (JSON-like, but binary encoded)
- Output_Action: variable
- Decision_Maker: 16 bytes (AI model ID หรือ Human ID)
- Override_Flag: 1 byte (0=AI, 1=Human, 2=Auto-escalation)
- Signature: 32 bytes (quantum signature for non-repudiation)
```

#### 4.3.2 Accountability Matrix
| Scenario                       | Responsible Party    | Accountable Party      | Audit Evidence                 |
| ------------------------------ | -------------------- | ---------------------- | ------------------------------ |
| AI decision caused delay       | AI Model (Developer) | Engineer               | Decision log + model version   |
| Human override caused incident | Human Operator       | Local Supervisor       | HITL log + override flag       |
| System failure due to bug      | Developer (Engineer) | Architect              | Code commit + test report      |
| Data breach                    | DevOps (security)    | Architect              | Access log + encryption status |
| Ethical violation              | All team             | Instructor (simulated) | Ethical review minutes         |


### 4.4 Governance and HITL Testing Plan
| Test Scenario | Description                                 | HITL Level Tested | Success Criteria                       | Owner      |
| ------------- | ------------------------------------------- | ----------------- | -------------------------------------- | ---------- |
| HITL-01       | Quantum sync fails, human approves fallback | L2                | Human approval received within timeout | Tester/QA  |
| HITL-02       | Human override during emergency             | L3                | System accepts manual command          | Tester/QA  |
| HITL-03       | No human response → auto-escalation         | L2 → L3           | Escalation happens after timeout       | Engineer   |
| HITL-04       | Dashboard display accuracy                  | L1                | All critical info shown correctly      | DevOps     |
| GOV-01        | Policy change propagation                   | Level 0→1→2       | All levels receive update              | Architect  |
| GOV-02        | Audit trail completeness                    | All               | Every decision has log entry           | Specialist |

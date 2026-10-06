Reimbursement Process Mining & Log Reliability Analysis
Overview
This project applies process mining to an employee reimbursement workflow to understand how claims actually move through the process, identify deviations and incomplete cases, and evaluate the quality and reliability of the underlying event log before analysis.
The project is deliberately structured in two stages:
Log Reliability Audit — determine whether the event log is trustworthy enough for process mining.
Process & Activity Analysis — use PM4Py to discover process flows, variants, bottlenecks, repetition/rework, waiting, cost-related patterns, and partial cases.
The project uses a small synthetic reimbursement event log containing 25 cases and 76 recorded events. Because the dataset is synthetic and small, the findings are intended to demonstrate the analytical methodology rather than make production-level business claims.
---
Business Problem
Reimbursement processes often look straightforward on paper:
Submit Claim → Manager Review → Finance Review → Approve Reimbursement → Payment
In practice, claims may:
stop before payment,
repeat an activity,
take different paths,
experience waiting between activities,
involve different resources or departments,
or contain data-quality problems that can distort process analysis.
Traditional reporting generally tells us how many claims were processed or the average processing time. Process mining adds another dimension: it reconstructs the actual sequence of activities from event-level data.
The key business questions addressed in this project are:
What does the actual reimbursement process look like?
What are the most common process variants?
Where do cases deviate from the expected flow?
Which activities are repeated and may indicate rework?
Where is time being spent or waiting introduced?
Which activities appear to be operational hotspots?
Which cases are incomplete or potentially still open?
Can the event log itself be trusted before using it for process analysis?
---
Dataset
Event Log
`Reimbursement_Event_Log_25Cases.csv`
The event log contains:
Field	Description
`CaseID`	Unique reimbursement claim/case identifier
`EventNo`	Sequence number of the event within a case
`Activity`	Process activity performed
`Timestamp`	Time at which the event occurred
`Resource`	Person/team/system performing the activity
`Department`	Employee department
`EmployeeID`	Employee associated with the claim
`ClaimAmount`	Reimbursement claim amount
`PreviousActivity`	Previous activity recorded for the case
`ActivityDurationMin`	Recorded activity duration in minutes
Dataset characteristics
25 cases
76 event records
5 major process activities
Multiple departments and resources
Both complete and incomplete/partial cases
Repeated activities are intentionally present to demonstrate rework/variation
Synthetic data created for analytical demonstration
The event log follows the basic process-mining structure of:
> **Case + Activity + Timestamp**
with additional case and event attributes available for deeper analysis.
---
Project Architecture
```text
                    Reimbursement Event Log
                              |
                              v
                  +------------------------+
                  | Log Reliability Audit  |
                  +------------------------+
                              |
             +----------------+----------------+
             |                                 |
             v                                 v
       Schema Validation                Data Quality
       Case/Activity/Time               & Freshness
             |                                 |
             +----------------+----------------+
                              |
                              v
                    Trusted Event Log
                              |
                              v
                    +------------------+
                    |      PM4Py       |
                    +------------------+
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
     Process Map          Variants           Bottlenecks
          |                   |                   |
          +-------------------+-------------------+
                              |
                              v
                  Activity-Level Analysis
                              |
       +----------+-----------+----------+-----------+
       |          |           |          |           |
       v          v           v          v           v
   Frequency   Duration    Waiting    Rework      Cost
                              |
                              v
                       Partial Cases
                              |
                              v
                       Activity Scorecard
```
---
1. Log Reliability Auditor
Before performing process mining, the project first asks:
> **Can we trust the event log?**
This is important because process mining is highly dependent on event-log quality. A visually attractive process map built from incorrect or incomplete event data can lead to incorrect business conclusions.
The reliability notebook performs several checks.
Supported input formats
The audit engine supports:
CSV
TSV
TXT
XLS/XLSX
JSON
XES
Parquet
Schema discovery
The auditor attempts to identify:
Case ID
Activity
Timestamp
Resource
Department
Case attributes
Event metadata
Column identification uses a combination of column-name patterns and data heuristics.
The user can manually confirm or override the automatically detected mandatory columns.
Mandatory-field validation
The audit checks whether the log contains valid:
Case identifiers
Activity labels
Timestamps
It also checks for issues such as:
missing case IDs
missing activities
missing timestamps
suspicious case-ID structures
weak schema-matching confidence
invalid timestamp values
unusually high numbers of activity labels
duplicated activity labels caused by casing or whitespace variations
Data-quality checks
The audit evaluates common event-log problems such as:
missing values
invalid timestamps
duplicate events
inconsistent activity labels
suspicious event ordering
case-level anomalies
Freshness assessment
The audit also evaluates the temporal coverage of the event log, including:
first event
last event
total time span
days since the last event
weeks with no events
future-dated events
Statistical profile
The auditor calculates structural characteristics such as:
number of events
number of cases
events per case
number of distinct activities
number of process variants
average case duration
median case duration
number of resources
Reliability score
The notebook combines multiple dimensions into an overall score:
Mandatory fields — 40%
Data quality — 25%
Attributes — 20%
Freshness — 15%
The score is mapped to qualitative grades such as:
Excellent
Good
Fair
Poor
Unusable
The purpose is not to claim that one universal score defines data quality. It provides a structured screening mechanism before process analysis.
---
2. Process Discovery with PM4Py
The second notebook uses PM4Py, an open-source Python library for process mining.
The event log is transformed into a structure where each case represents a process instance and each event represents an activity occurring at a particular timestamp.
Directly-Follows Graph
A Directly-Follows Graph (DFG) is generated to visualize how activities actually follow one another in the event log.
For example:
```text
Submit Claim
      |
      v
Manager Review
      |
      v
Finance Review
      |
      v
Approve Reimbursement
      |
      v
Payment
```
The important point is that this flow is discovered from observed event data, rather than manually drawing the process.
The PM4Py notebook uses process discovery to generate the DFG and a performance-oriented DFG.
---
3. Process Variants
A process variant is a distinct sequence of activities followed by one or more cases.
For example:
```text
Variant A:
Submit → Manager Review → Finance Review → Approve → Payment

Variant B:
Submit → Manager Review → Finance Review

Variant C:
Submit → Manager Review → Manager Review → Finance Review → Finance Review
```
Variant analysis helps identify:
the common/happy path
alternative paths
incomplete cases
repeated activities
process deviations
This is particularly useful because averages can hide process variation.
---
4. Bottleneck / Performance Analysis
The project also creates a performance-oriented process view.
Instead of looking only at how frequently an activity occurs, the performance view considers where time is being consumed.
This allows questions such as:
Which transitions take longer?
Where is waiting introduced?
Which activities deserve investigation first?
A critical interview distinction is:
> **The most frequent activity is not necessarily the bottleneck.**
A bottleneck should be investigated using time, queue/waiting behavior, capacity, repetition, and process context rather than frequency alone.
---
5. Formal Process Model
The PM4Py notebook also demonstrates discovery of a Petri net using inductive process discovery.
A Petri net represents the process more formally using:
places/states
transitions/activities
tokens representing process state
This representation can support more rigorous process analysis and conformance checking.
For a business audience, however, the DFG is generally easier to interpret.
---
6. Activity-Level Analysis
The activity-level notebook goes deeper than simply drawing a process map.
It evaluates six major dimensions:
1. Frequency
Measures:
number of executions
percentage of cases containing the activity
executions per case
share of total events
2. Duration
Where appropriate timestamp information is available, the analysis considers:
mean
median
P90
maximum
total duration
variability
3. Waiting
The analysis separates:
waiting before an activity
waiting caused after an activity
expensive handovers
This distinction is important because a long activity and a long queue are different operational problems.
4. Repetition / Rework
The analysis identifies:
activities performed more than once
rework rate
extra executions
self-loops
A self-loop is an immediate repetition such as:
```text
Finance Review → Finance Review
```
A non-adjacent repetition can indicate a larger rework loop.
5. Cost
The framework can use a cost field from the event log or estimate cost using:
```text
Estimated Cost
= Fixed Cost per Execution
+ Hourly Rate × Processing Time
```
Rework cost can then be separated from first-time execution cost.
6. Partial Cases
Cases are classified as partial when they do not satisfy the expected start/end conditions.
The analysis identifies:
partial-case rate
where cases stopped
activities associated with partial cases
lift of partial-case probability
potential drop-off points
A useful interpretation is:
> An activity with lift greater than 1 is associated with a higher probability of a case being partial than the average case.
This is an association, not proof of causation.
---
7. Activity Scorecard
The activity scorecard brings the analysis together.
Activities are ranked across dimensions such as:
frequency
duration
waiting before the activity
waiting caused by the activity
rework
cost
partial-case association
A combined hotspot score helps prioritize which activities deserve further investigation.
The purpose is prioritization—not declaring that the highest score is automatically the root cause.
---
8. Key Analytical Insights
Because the dataset is synthetic and contains only 25 cases, the project should be presented as a methodology demonstration rather than a statistically representative operational study.
The event log deliberately contains:
complete cases
incomplete cases
repeated activities
multiple process variants
different resources
multiple departments
This makes it possible to demonstrate how process mining can uncover behavior that a simple aggregate report would miss.
One useful observation from the supplied log is that several cases stop before the final `Payment` activity, while some cases contain repeated `Manager Review` or `Finance Review` events. These patterns are exactly the type of behavior that variant and activity-level analysis is designed to surface. citeturn2view0turn4view0
---
Technology Stack
Python
PM4Py
Pandas
Jupyter Notebook / Google Colab
IPyWidgets
Graphviz
Plotly for interactive activity-level analysis
The PM4Py notebook includes DFG discovery, performance DFG discovery, process variants, resource analysis and Petri-net discovery. citeturn2view1turn4view1turn4view2
The activity-level analysis includes frequency, duration, waiting, repetition/rework, cost, partial-case analysis, scorecards and path comparison. citeturn2view2turn3view4turn3view5turn3view6
---
Repository Structure
```text
reimbursement-process-mining-proj/
│
├── Reimbursement_Event_Log_25Cases.csv
│
├── Log_Reliability_Auditor_Workshop.ipynb
│
├── PM4Py_Process_Mining_Workshop (2).ipynb
│
├── activity_level_analysis_pm4py_(1).ipynb
│
└── README.md
```
---
How to Run
Option 1 — Google Colab
Google Colab is the easiest option.
Open Google Colab.
Upload the required `.ipynb` notebook.
Upload `Reimbursement_Event_Log_25Cases.csv`.
Run the notebook from top to bottom.
Select the uploaded event log when prompted.
Confirm the Case ID, Activity and Timestamp columns.
Explore the generated analysis.
Option 2 — Local Jupyter Environment
Install the required packages:
```bash
pip install pandas pm4py ipywidgets graphviz plotly openpyxl
```
You may also need the Graphviz system executable, not just the Python package, for process-map rendering.
Then open the notebooks using Jupyter:
```bash
jupyter notebook
```
---
Recommended Analysis Workflow
The intended workflow is:
```text
1. Load event log
        ↓
2. Validate schema
        ↓
3. Check data quality
        ↓
4. Check freshness / completeness
        ↓
5. Establish trusted event log
        ↓
6. Discover process map
        ↓
7. Analyse process variants
        ↓
8. Analyse bottlenecks
        ↓
9. Analyse activity frequency
        ↓
10. Analyse waiting and duration
        ↓
11. Identify rework
        ↓
12. Analyse cost
        ↓
13. Analyse partial cases
        ↓
14. Prioritize hotspots
        ↓
15. Recommend process improvements
```
---
Business Value
Process mining can help organizations move from:
> **"We think this process is slow."**
to:
> **"The event data shows where cases deviate, where waiting occurs, which activities repeat, and which parts of the process should be investigated first."**
For reimbursement operations, this can support improvements such as:
reducing unnecessary review loops
improving first-time-right submissions
identifying approval delays
improving workload allocation
reducing incomplete cases
improving process compliance
prioritizing automation opportunities
---
Important Limitations
This project has several limitations that should be explicitly acknowledged in an interview:
The dataset is synthetic.
The dataset contains only 25 cases and 76 events.
Small samples should not be used to make strong statistical claims about a real organization.
A single timestamp per event does not always allow a perfect separation between processing time and waiting time.
Correlation or association discovered through process mining does not establish causation.
A process map shows observed behavior; it does not automatically explain why that behavior occurred.
Cost estimates depend on assumptions such as hourly labor rates and fixed execution costs.
A partial case may be an ongoing case rather than a failure.
These limitations are important because a good analyst should distinguish what the data proves from what the analyst suspects.
---
Future Enhancements
Potential next steps include:
Add a larger real-world event log.
Add automated conformance checking against a predefined reimbursement BPMN.
Calculate process cycle time and SLA compliance.
Add statistical significance testing for resource/department differences.
Add root-cause analysis for repeated activities.
Add a Streamlit dashboard.
Add automated anomaly detection.
Add approval-level workload analysis.
Add what-if simulation for process improvements.
Add automated recommendations based on identified bottlenecks.
Integrate the output with Power BI for stakeholder reporting.
---
Interview Summary
One-line project description
> **Built a process-mining analysis for an employee reimbursement workflow using PM4Py, combining event-log reliability auditing, process discovery, variant analysis, bottleneck analysis, rework detection and activity-level performance analysis.**
30-second explanation
> I worked on a process-mining project for an employee reimbursement process. I started with an event log containing case IDs, activities and timestamps, but instead of directly building a process map, I first built a log-reliability audit to check whether the data was trustworthy. Then I used PM4Py to discover the actual process flow, identify process variants and performance bottlenecks. I further analysed individual activities for frequency, waiting, repetition, cost and partial cases. The main objective was to move from simply visualizing the process to identifying where deviations, delays and rework could be investigated for process improvement.
90-second explanation
> The problem I was trying to solve was understanding how an employee reimbursement process actually operates rather than relying only on the documented process or aggregate KPIs.
>
> I used an event log where each reimbursement claim is a case and each activity has a timestamp, resource and additional attributes. The first stage was a log reliability audit because process mining is only as reliable as the event data. I checked mandatory fields such as Case ID, activity and timestamp, along with data-quality issues, schema consistency and freshness.
>
> Once the log was considered usable, I used PM4Py to generate a Directly-Follows Graph and a performance view of the process. I also analysed process variants to understand the different sequences followed by claims.
>
> I then went deeper into activity-level analysis. I looked at frequency, duration, waiting before and after activities, repeated activities as potential rework, cost, and partial cases. I also created an activity scorecard to prioritize potential hotspots.
>
> One important learning from the project was that a process-mining map doesn't automatically tell you the root cause. It tells you where the behavior is occurring. You still need business context and further analysis to determine why it is happening and what intervention makes sense.
---
Core Learning
The main takeaway from the project is:
> **Don't start process improvement with the process map. Start by asking whether the event log is trustworthy, then discover the process, quantify deviations, and only then investigate root causes and improvements.**

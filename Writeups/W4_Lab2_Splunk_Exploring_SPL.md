# Writeup: Splunk - Exploring SPL (TryHackMe)

**Room Link:** [https://tryhackme.com/room/splunkexploringspl](https://tryhackme.com/room/splunkexploringspl)
**Difficulty:** Medium
**Status:** Completed (100%)

---

## Overview
This room focuses on Splunk's Search Processing Language (SPL). It covers basic search operators, filtering, structuring data, transforming commands (stats, chart, timechart), data enrichment (iplocation, lookup, eval), and using SPL for basic anomaly detection.

---

## Task 1: Introduction
**Objective:** Understand the learning objectives and start the lab machine.

*   **Question:** I am ready to explore Splunk's query language!
*   **Answer:** `No answer needed`

---

## Task 2: Search & Reporting
**Objective:** Familiarize with the Search & Reporting app interface and perform a basic search using the `windowslogs` index.

*   **Question:** Submit your first query for All Time: `index=windowslogs`. How many total events do you see?
*   **Answer:** `12256`
*   **Question:** After you submit your first query, look in the Fields sidebar. Which **SourceIP** has recorded the most amount of events?
*   **Answer:** `172.90.12.11`
*   **Question:** How many events appear on 04/15/2022 from 08:05 AM to 08:06 AM?
*   **Answer:** `134`

---

## Task 3: Search Operators
**Objective:** Learn how to use relational, logical, and wildcard operators to refine searches.

*   **Question:** How many events in the `windowslogs` index have an **EventID** field value equal to `4624`?
    *   *Query:* `index=windowslogs EventID=4624`
*   **Answer:** `26`
*   **Question:** How many events are observed with the **DestinationIp** = `172.18.39.6` and **DestinationPort** = `135`?
    *   *Query:* `index=windowslogs DestinationIp=172.18.39.6 DestinationPort=135`
*   **Answer:** `4`
*   **Question:** Use the query `index=windowslogs Hostname=Salena.Adam DestinationIp=172.18.38.5`. Which **SourceIp** returns the highest count?
*   **Answer:** `172.90.12.11`
*   **Question:** How many events are returned when you search the term `cyber*`?
    *   *Query:* `index=windowslogs cyber*`
*   **Answer:** `12256`
*   **Question:** Which operator is given the lowest priority in Splunk searches?
*   **Answer:** `AND`

---

## Task 4: Filtering Results
**Objective:** Use commands like `fields`, `dedup`, and `regex` to filter out noise and find specific data points.

*   **Question:** Use the `fields` command to highlight `Domain`, `SourceProcessId`, and `TargetProcessId`. Which **SourceProcessId** has the highest value?
*   **Answer:** `9496`
*   **Question:** Try out this query: `index=windowslogs | regex TargetObject="Manager$"`. Which **TargetObject** field value contains the highest number of results?
*   **Answer:** `HKLM\SOFTWARE\Microsoft\SecurityManager`

---

## Task 5: Structuring Results
**Objective:** Use the `table` command and sorting commands (`head`, `tail`, `sort`, `reverse`) to build readable timelines.

*   **Question:** Build a table that highlights the `EventID`, `AccountName`, and `AccountType` fields. Which **AccountName** appears first in your results?
    *   *Query:* `index=windowslogs | table EventID AccountName AccountType`
*   **Answer:** `SYSTEM`
*   **Question:** Append the above query to include the `reverse` command. Which **EventID** appears first?
    *   *Query:* `index=windowslogs | table EventID AccountName AccountType | reverse`
*   **Answer:** `800`
*   **Question:** Use the query below to build a timeline of events. What password was given to the user `A1berto`?
    *   *Query:* `index=windowslogs EventID=1 | table _time ParentProcessId ProcessId ParentCommandLine CommandLine | reverse`
*   **Answer:** `paw0rd1`

---

## Task 6: Transforming Commands
**Objective:** Learn how to use aggregating and statistical commands like `top`, `rare`, `stats`, `chart`, and `timechart`, as well as data enrichment commands (`iplocation`, `lookup`, `eval`).

*   **Question:** Use the `top` command to query the **Image** field. Which **Image** field value has the most occurrences?
    *   *Query:* `index=windowslogs | top Image`
*   **Answer:** `C:\windows\system32\svchost.exe`
*   **Question:** Try out the `iplocation` command with the **SourceIp** field. Which **Region** do the IP addresses in your events originate from?
    *   *Query:* `index=windowslogs | iplocation SourceIp | stats count by Region`
*   **Answer:** `California`
*   **Question:** Try out this `lookup` query. Which **Image** field value has the highest **RiskScore**?
    *   *Query provided in room:*
        ```spl
         index=windowslogs 
         | lookup image_riskscore Image OUTPUT RiskScore 
         | stats count by Image RiskScore 
         | sort - RiskScore
        ```
   
  * Answer: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
    
### Task 7: Anomaly Detection
 * Objective: Use SPL to identify outliers in datasets  by calculating frequencies and standard deviations (z-scores).

 * Question: Run the first anomaly detection query (index=vpnlogs). Which other user is marked as an outlier?

 * (Based on the screenshot, the first query filters by country_freq < 0.1)

* Answer: jsmith

* Question: Which country is anomalous for the user from Q1?

* Answer: JP

* Question: Run the second anomaly detection query (index=vpnlogs). Which user suspiciously logged in at 3 AM?

* (Based on the screenshot, the second query calculates a zscore > 3 for login hours)

* Answer: njackson

### Task 8: Conclusion
 * Objective: Complete the room.

* Question: Complete the room and continue on your cyber learning journey!

* Answer: No answer needed



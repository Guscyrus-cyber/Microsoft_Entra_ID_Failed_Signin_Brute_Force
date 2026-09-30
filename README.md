Microsoft Entra ID Failed Sign-In & Brute-Force Investigation\
**Introduction**
--------------------------------------------------------------

Failed sign-ins are among the most common authentication events monitored by Security Operations Center (SOC) analysts. Although many failures are caused by legitimate users entering incorrect credentials, repeated authentication failures can also indicate malicious activity such as brute-force attacks, password spraying, credential stuffing, or attempts to access compromised accounts.

Microsoft Entra ID provides detailed sign-in logs that allow SOC analysts to examine authentication activity, including the affected user, source IP address, geographic location, application, authentication method, failure reason, error code, and Conditional Access information. These details help distinguish normal authentication mistakes from potentially suspicious behavior.

This lab provides hands-on experience with Tier 1 authentication triage by generating controlled failed sign-in activity and investigating the resulting events through Microsoft Entra ID. The investigation focuses on recognizing repeated authentication failures, analyzing their source and characteristics, identifying suspicious patterns, and determining whether an event should be documented, monitored, or escalated for further investigation.
\
**Purpose:** Learn how a Tier 1 SOC analyst identifies and investigates repeated authentication failures in Microsoft Entra ID and determines whether the activity represents a simple user mistake or potentially malicious brute-force/password-spraying behavior.

The main workflow will be:

Generate failed sign-ins → Detect repeated failures → Investigate user/IP/error codes → Identify the pattern → Determine suspicious vs. benign → Correlate with Sentinel → Document and escalate if necessary.\
\
**Step 1 —** Using the administrator as the investigator and the SOC Test User to safely generate the authentication events. (First image )\
The Microsoft Entra admin center is open correctly under the Global Administrator account**.** This account will serve as the SOC analyst/investigator, while the SOC Test User will generate the authentication activity for investigation.\
\
Establish the Baseline\
Before generating brute-force activity, first reviewing the current sign-in logs. This provides a baseline so that newly generated failed authentication attempts can be distinguished from existing events.

I click User sign-ins. 
The sign-in log is displaying correctly. The existing activity provides the baseline before generating the new brute-force pattern.

### An important existing event is already visible:
SOC Test User → My Profile → Failure → Error 50126\
That is an older single failed-password event from the previous lab I did. It should not be confused with the new events that will be generated for this lab. 

Step 2 — Generate Repeated Failed Sign-Ins

The next objective is to create several controlled failed authentication attempts against the SOC Test User. This will simulate the pattern that a Tier 1 SOC analyst might encounter during brute-force triage.\
I Open a Private/Incognito browser window separately. 
I Generate the First Failed Sign-In.\
[Microsoft My Account](https://myaccount.microsoft.com/?utm_source=chatgpt.com)\
I enter my **SOC Test User** account. 

Step 3 — Controlled Failed Authentication

 I Click Next.
At the password screen:\
I Enter an intentionally **incorrect password**.\
I Submit it **once only**.\
Failed Sign-In Generated\
The Microsoft authentication interface returned:\
“Your account or password is incorrect".
This establishes the user-side evidence that authentication failed because invalid credentials were submitted. 

Step 4 — Investigate the Event in Microsoft Entra ID

Now I return to the administrator Entra ID window**.\**
Entra ID → Users → Sign-in logs

Looking for a new entry for:\
**User:** SOC Test User\
**Status:** Failure\
Now, The new controlled event is now visible at the top: 

**8/23/2026, 9:46:11 PM — SOC Test User — My Profile — Failure — Error 50126**

This confirms that the failed authentication generated in Chrome Incognito was recorded by Microsoft Entra ID.

Step 5 — Open the Failed Event

I Click the top Failure row — the event at 9:46:11 PM**.\**
The Activity Details: Sign-ins panel opens.**\**
I start with basic Sign-In Investigation**\**
The Basic info confirms the controlled authentication failure:

**User:** SOC Test User\
**Username:** testuser01@liberty566yahoo.onmicrosoft.com\
**Status:** Failure

### **Sign-in error code:** **50126**
**Failure reason:** Error validating credentials due to invalid username or password.\
**Authentication requirement:** Single-factor authentication\
**Time:** August 23, 2026 at 9:46:11 PM\
**Error 50126** indicates that Microsoft Entra ID received the authentication request, but the supplied credentials were invalid. At this point, a single occurrence is consistent with an incorrect-password attempt and does not by itself establish brute-force activity.

Step — 6: Source Investigation

### I Click **Location** at the top.
Question: **Where did the failed authentication originate, and what source IP address was associated with it?\
Location:** Lorton, Virginia, US\
**Source IP address:** 108.28.79.19\
**Autonomous System Number (ASN):** 701\
**Global Secure Access:** No\
**Named location:** None configured\
The failed sign-in originated from **IP address 108.28.79.19**, geolocated by Microsoft Entra ID to **Lorton, Virginia, US**. 

This information become important when correlating multiple authentication failures. Repeated failures from the same IP against one account can indicate password guessing, while one IP attempting authentication against many accounts can indicate password spraying.

Step 7 — Device Investigation

I Click Device info.
Question: What device, operating system, and browser generated the failed authentication attempt?
The Device info confirms the client environment associated with the failed authentication:

**Browser:** Chrome 151.0.0\
**Operating system:** macOS\
**Compliant:** No (Entra ID does not have a positive compliance state for this Mac for the sign-in. In a managed environment, compliance policies can require things such as encryption, acceptable OS versions, passwords, or other security settings. So, it means Entra ID is not reporting it as compliant with the organization's device-compliance requirements for this event).\
**Managed:** No (Microsoft Entra ID does not see this Mac as being under the organization's device-management system for this sign-in. In a Microsoft environment, device management is commonly performed through Intune/MDM. A managed corporate Mac might have organizational security settings, configuration profiles, required applications, and other controls applied to it).\
**Device ID:** Not reported\
**Join type:** Not reported

The failed authentication originated from a **Chrome browser on macOS**. Entra ID reports the endpoint as unmanaged and non-compliant with no registered Device ID or join type.
This does not automatically make the event malicious. However, device state becomes useful during triage because an authentication attempt from an unknown or unmanaged endpoint may require additional investigation when combined with other indicators such as repeated failures, unusual IP addresses, impossible travel, or successful authentication after multiple failures. 

Step 8 — Authentication Investigation

I Click on Authentication Details.
Question: **What authentication method was attempted, did it succeed, and what specifically caused the authentication failure?\**
The **Authentication Details** tab was reviewed to determine which authentication method was attempted and whether authentication succeeded.

**Authentication method:** Password\
**Authentication method detail:** Password in the cloud\
**Succeeded:** False\
**Result detail:** Invalid username or password

The authentication attempt failed during password validation. Microsoft Entra ID rejected the credentials because the supplied username/password combination was invalid.

This is important during SOC triage because repeated authentication failures—particularly from the same IP address or against the same account—can indicate password guessing, brute-force activity, or credential-based attacks rather than an isolated user mistake.
Finding: Password authentication attempted → Authentication failed → Invalid credentials

Triage Scope and Transition to Brute-Force Investigation

A SOC analyst does not need to examine every available Microsoft Entra ID field during the initial triage of every authentication event. The investigation should focus on the information most relevant to determining what occurred and whether the activity represents a security concern.

For this lab, representative triage was performed using Basic Info, Location, Device Info, Authentication Details, and Conditional Access. These fields provided sufficient information to identify the failed authentication, determine the failure reason, examine the source and endpoint context, and understand the authentication behavior.

Additional fields and deeper investigation can be examined when an event is suspicious or requires escalation. With the fundamental failed sign-in triage completed, the lab now proceeds to brute-force investigation**,** where multiple authentication failures will be analyzed as a pattern rather than as an isolated event.

Step 9 — Brute-Force Authentication Investigation

The next stage examines how **multiple failed sign-in attempts** can indicate a possible brute-force attack. Unlike a single failed login, which may simply result from an incorrect password, repeated authentication failures against the same account within a short period can represent suspicious activity. For this controlled lab, several incorrect password attempts will be generated against the **SOC Test User** account. Microsoft Entra ID sign-in logs will then be used to investigate the resulting pattern.

The investigation will focus on the number and timing of failures, username, source IP address, location, authentication result, and sign-in error codes**.** The objective is to determine whether the sequence resembles brute-force behavior and understand how a Tier 1 SOC analyst would identify, document, and potentially escalate such activity.

I Generate the controlled failed sign-in attempts.Using **SOC Test User** (testuser01@...onmicrosoft.com).\
Five controlled authentication attempts were performed against the **SOC Test User** account using incorrect passwords within a short period. Each attempt was rejected by Microsoft authentication with an incorrect account/password message.\
The repeated failures were intentionally generated to simulate the authentication pattern that may occur during a **password-guessing or brute-force attack**. The next stage of the investigation is to determine how these attempts appear from the SOC analyst's perspective in Microsoft Entra ID.\
Brute-Force Simulation Completed. 

Step 10 — Review Repeated Failed Sign-Ins\
After the controlled authentication attempts were completed, the administrator session was used to return to **Microsoft Entra ID → Users → Sign-in logs** and I refresh the authentication records.\
The sign-in logs now show **five consecutive failed authentication attempts** against the **SOC Test User**:

**11:11:54 PM** — Failure — Error **50126**\
**11:12:44 PM** — Failure — Error **50126**\
**11:13:06 PM** — Failure — Error **50126**\
**11:13:24 PM** — Failure — Error **50126**\
**11:13:42 PM** — Failure — Error **50126**

All five events target the same **SOC Test User**, use the **My Profile** application, and return **error code 50126**, representing invalid username or password credentials.\
This is significantly more interesting to a SOC analyst than a single isolated failed login because the events form a repeated authentication-failure pattern within approximately two minutes. In a production environment, this pattern could indicate password guessing or attempted brute-force activity and would justify further investigation. 


Step 11 — Correlation of the Brute-Force Attempts

After identifying five repeated authentication failures, the next stage is to determine whether the events are related. A SOC analyst correlates common attributes such as **source IP address, geographic location, device/browser, username, application, timestamps, and authentication result**.

### Investigation
I Select the first of the five recent Failure events — for example, the **11:13:42 PM** event.\
I Open the event and review:

**Basic info**

Confirm **Status = Failure**\
Confirm **Sign-in error code = 50126**\
Confirm **User = SOC Test User** 

Then I open **Location**.
The key information needed at this point is:\
IP address + Location
This confirms the first event in the sequence:

**User:** SOC Test User

**Status:** Failure\
**Error code:** 50126\
**Failure reason:** Invalid username or password\
**Authentication:** Single-factor authentication\
**Source IP:** 108.28.79.19\
**Location:** Lorton, Virginia, US\
**Time:** 8/23/2026, 11:13:42 PM 

This event matches the authentication failure pattern observed during the simulated brute-force activity.
A single 50126 event normally represents an incorrect credential attempt and, by itself, does not establish brute-force activity. However, the sign-in log previously showed five authentication failures against the same SOC Test User within approximately two minutes. The repeated failures within a short period create the important indicator. Correlation of the events by user account, timestamps, error code, source IP, and location allows the activity to be classified as a simulated brute-force pattern rather than an isolated password mistake.

Step 12 — Verify the Common Source

I Return to the sign-in log, and I open one of the other four recent failed events**.** I Select Location and compare its IP address with: 108.28.79.19\
If the second event also shows 108.28.79.19, that will provide direct evidence that multiple authentication attempts originated from the same source IP**.\**
Yes, this confirms that another failed authentication event shows the same source IP address: 108.28.79.19 and the same location, Lorton, Virginia, US.

Step 13 — Source IP Correlation

Multiple failed authentication events associated with the SOC Test User were correlated using their source information. The investigated events originated from the same IP address, 108.28.79.19, and the same geographic location.

Combined with the five 50126 credential failures occurring within a short time window, this strengthens the identification of the activity as a simulated brute-force authentication pattern.

Step 14 — Device Correlation

For this same failed event, I select Device info**.** The objective is to determine whether the repeated attempts also share the same browser, operating system, management status, and compliance status.

The Device info confirms the authentication attempt originated from:

**Browser:** Chrome 151.0.0\
**Operating System:** macOS\
**Compliant:** No

**Managed:** No\
**Device ID:** None recorded\
**Join Type:** None recorded

The event originated from an unmanaged and non-compliant macOS endpoint. No Device ID or Entra join type is associated with the authentication attempt.
When correlated with the previous findings, multiple **50126** authentication failures against the same account from the same source IP within a short period, the device information provides another useful contextual indicator for the investigation. 

Step 15 — Authentication Correlation

I Select Authentication Details for this same event. The next check is whether the attempt again shows Password in the cloud, Succeeded: false, and invalid username or password.

The Authentication Details confirm:

**Authentication method:** Password\
**Authentication detail:** Password in the cloud\
**Succeeded:** false\
**Result:** Invalid username or password\
**Time:** 8/23/2026, 11:12:44 PM

This confirms that the authentication attempt failed during password validation. Combined with the other correlated events, the investigation now shows repeated password failures against the same SOC Test User from the same source IP, over a short period. The evidence therefore supports a simulated brute-force authentication pattern. (Image 22)

Step 16 — Conditional Access Check

I Select Conditional Access. This determines whether any Conditional Access policy evaluated or affected this failed authentication attempt.

The Conditional Access tab shows: Policy Name: Not applicable\
No Conditional Access policy was applied to this failed authentication event. The authentication failure occurred because the submitted credentials were invalid, rather than because access was blocked by a Conditional Access policy.\
At this point, the investigation has established the important evidence: repeated failures, error code **50126**, same account, same source IP/location, device context, failed password authentication, and Conditional Access status.


Step 17 — SOC Analyst Investigation Conclusion

The investigation identified five failed authentication attempts against the SOC Test User within Step 16 — SOC Analyst Investigation Conclusion

The investigation identified five failed authentication attempts against the SOC Test User within a short time period. Event correlation showed the following consistent indicators:

Multiple authentication failures against the **same user account**\
Sign-in error code **50126**\
Failure reason: **invalid username or password**\
Same source IP address: **108.28.79.19**\
Same geographic location: **Lorton, Virginia, US**\
Authentication method: **Password / Password in the cloud**\
Authentication result: **Failed**\
Endpoint: **macOS using Chrome**\
Device reported as **unmanaged and non-compliant**\
No applicable **Conditional Access policy**

The concentration of repeated credential failures against the same account from the same source within a short period is consistent with a brute-force authentication pattern. In this controlled lab, the activity was intentionally generated and therefore represents a simulated brute-force attack rather than an actual security incident**.** In a production SOC environment, comparable unexplained activity would require additional correlation, assessment of subsequent successful logins, source-IP reputation and historical activity, and escalation when compromise or continued malicious authentication activity is suspected.

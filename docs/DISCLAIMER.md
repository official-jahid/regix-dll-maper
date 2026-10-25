# docs/DISCLAIMER.md — Full Legal, Authorization & Rules of Engagement

> ⚖️ This document contains the complete legal and authorization framework for `regix-dll-maper`.
> Read it **before** using the tool.

---

## 🚨 AUTHORIZED USE ONLY

**regix-dll-maper** is an offensive-security style PowerShell toolkit. It:

- Stops Windows services
- Disables PowerShell logging (ModuleLogging, ScriptBlockLogging)
- Deletes/truncates Windows Event Log files
- Hides the console window
- Downloads and manually maps a DLL into the current process
- Clears PowerShell command history

**You may only use this tool on systems that you own or for which you have obtained explicit, written authorization to perform security testing.**

Unauthorized use may violate laws including (but not limited to):

- **USA:** Computer Fraud and Abuse Act (CFAA), state computer crime laws
- **UK:** Computer Misuse Act 1990
- **EU:** Various national cybercrime laws implementing EU directives; GDPR-related provisions

---

## ✍️ Written Permission Requirement

Before running `regix-dll-maper`, obtain written authorization that specifies:

| Element                     | Requirement                                                                   |
| --------------------------- | ----------------------------------------------------------------------------- |
| **System owner**            | Name, organization, contact details                                           |
| **Target systems**          | Hostnames, IP addresses, or asset tags in scope                               |
| **Time window**             | Start and end dates/times for testing                                         |
| **Techniques permitted**    | Explicitly list: service stop, log suppression, DLL mapping, event log wiping |
| **Approver**                | Authorized representative with sign-off authority                             |
| **Authorization reference** | Ticket number, contract ID, or memo reference                                 |

**Keep this authorization on file for the duration of testing and for audit retention.**

---

## 🌍 Per-Jurisdiction Notes

> These notes are **not legal advice**. Consult qualified legal counsel in your jurisdiction before testing.

| Jurisdiction       | Key Consideration                                                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| **United States**  | CFAA criminalizes unauthorized access. Written authorization is your primary defense. State laws may add requirements. |
| **United Kingdom** | Computer Misuse Act 1990 sections 1–3 cover unauthorized access, modification, and impairment.                         |
| **European Union** | National laws implement cybercrime directives. GDPR may apply if personal data is on affected systems.                 |
| **Canada**         | Criminal Code provisions on unauthorized use of computers.                                                             |
| **Australia**      | Cybercrime Act 2001 (Cth).                                                                                             |
| **Other**          | Consult local counsel.                                                                                                 |

**When in doubt: do not run the tool. Seek legal advice first.**

- **Other jurisdictions:** Local computer misuse, unauthorized access, and data protection laws

---

## ❌ No Warranty / No Liability

`regix-dll-maper` is provided under the **MIT License**:

> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**You assume all risk.** The author is not responsible for:

- System instability or crashes
- Data loss from event log deletion
- Legal consequences of unauthorized use
- Detection by EDR/AV and subsequent incident response
- Any other direct, indirect, incidental, or consequential damages

---

## 📋 Rules of Engagement (RoE) Template

Use this template to scope authorized testing. Fill in all sections before running the tool.

```
================================================================
RULES OF ENGAGEMENT — regix-dll-maper
================================================================

Client / System Owner: _______________________________________
Authorized Tester:     _______________________________________
Authorization Ref:     _______________________________________
Date:                  _______________________________________

1. IN-SCOPE SYSTEMS
   Hostnames/IPs: ___________________________________________
   Asset tags:    ___________________________________________

2. OUT-OF-SCOPE SYSTEMS
   (Must NOT be tested) ______________________________________

3. PERMITTED TECHNIQUES
   [ ] Service stop (WSearch, cbdhsvc, VSS, fhsvc)
   [ ] PowerShell logging suppression (ModuleLogging, ScriptBlockLogging)
   [ ] Temp folder cleanup
   [ ] Hidden console execution
   [ ] Remote DLL download and manual mapping
   [ ] Event log wiping (System, PowerShell, Security)
   [ ] PowerShell history clearing

4. TIME WINDOW
   Start: _______________  End: _______________
   Timezone: ____________

5. NOTIFICATION REQUIREMENTS
   SOC / IR team notified: [ ] Yes  [ ] No
   Emergency contact: _______________________________________

6. DATA HANDLING
   Event logs will be: [ ] Preserved  [ ] Collected to SIEM  [ ] Other: _______

7. SIGN-OFF
   Owner signature: ___________________  Date: _______
   Tester signature: ___________________  Date: _______
================================================================
```

---

## 🔍 Pre-Run Confirmation Checklist

Complete every item before executing `.\main.ps1`:

- [ ] I own this system **OR** I have written authorization to test it
- [ ] The authorization specifies the systems, techniques, and time window
- [ ] The system is isolated or testing will not impact production operations
- [ ] Event logs are being forwarded to a SIEM (so local wiping doesn't destroy evidence)
- [ ] I understand the tool disables PowerShell logging and wipes event logs
- [ ] I understand the tool downloads and executes a remote DLL
- [ ] I have a plan to restore services and re-enable logging after testing
- [ ] I have reviewed `docs/DETECTION.md` to understand defender visibility
- [ ] I have consulted legal counsel if uncertain about jurisdiction

**If you cannot check every box, STOP. Do not run the tool.**

---

## 📬 Responsible Disclosure

If you discover vulnerabilities or defensive gaps while using this tool in authorized testing:

1. Report findings to the system owner / organization through their established channel.
2. Do not disclose publicly without the owner's permission.
3. Follow responsible disclosure timelines (typically 90 days for vendor notification).

---

## 📄 License Summary

`regix-dll-maper` is licensed under the MIT License (see `LICENSE`). The license grants broad rights to copy, modify, and distribute, **but does not authorize illegal or unethical use**. This disclaimer supplements — not replaces — the MIT License.

---

**Last updated:** 2026-10-25  
**Version:** 1.0.0

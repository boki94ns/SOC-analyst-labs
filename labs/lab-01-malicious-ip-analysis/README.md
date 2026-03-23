# 🛡️ Lab 01 - Malicious IP Communication Analysis

## 📌 Alert Overview
- Source IP: 10.2.4.33 (Internal)
- Destination IP: 192.243.59.20 (External)
- Device: Sophos Firewall
- Event Type: Proxy / Web Traffic
- Status Code: 200 (Successful connection)

---

## 🔍 Analysis

The alert indicates that an internal host initiated an outbound connection to an external IP address via web traffic.

Since the connection was successful (status code 200), the communication was allowed and completed.

This behavior may represent:
- Legitimate user activity (web browsing)
- Access to a potentially malicious website
- Possible malware communication (Command and Control - C2)

---

## 🌐 Threat Intelligence

### AbuseIPDB
- The IP address has been reported multiple times
- Associated with:
  - Malware activity
  - Phishing campaigns

This suggests a history of potentially malicious usage.

---

### VirusTotal
- The domain linked to the IP shows low detection rates
- However, some vendors classify it as suspicious or spam-related

---

### SecurityTrails
- The domain is associated with hosting infrastructure
- This may indicate shared hosting, which can sometimes be abused

---

## ⚠️ Conclusion

This is an **outbound connection** from an internal host to an IP address with a suspicious reputation.

Although there is no strong evidence of compromise, threat intelligence indicates potential risk.

The activity should be classified as **suspicious but not confirmed malicious**, and requires further monitoring.

---

## 🧯 Recommendation

- Monitor host: 10.2.4.33 for any additional suspicious activity
- Review user browsing behavior and access logs
- Perform an endpoint scan to verify if the system is compromised
- Investigate for repeated connections to the same IP address

If repeated connections are observed, especially within short time intervals, this may indicate automated communication such as malware beaconing.

In that case, blocking the IP address or domain should be considered.

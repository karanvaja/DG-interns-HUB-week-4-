# Custom Wazuh Rule

- **Rule ID:** 100051
- **Level:** 10
- **Description:** Custom Rule: Hack attempt detected!
- **Tested:** Yes, successfully verified using `wazuh-logtest`.

## Rule Code (XML)
```xml
<group name="local,syslog,sshd,">
  <rule id="100051" level="10">
    <match>hack_attempt</match>
    <description>Custom Rule: Hack attempt detected!</description>
  </rule>
</group>
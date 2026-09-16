# Splunk Searches

SPL used for triage in this lab. Each search should say what question it answers.

## Alert volume by signature
Answers: which detections are generating the most noise, and are any of them worth tuning?

```spl
index=suricata sourcetype=suricata:eve event_type=alert
| stats count by alert.signature, alert.signature_id
| sort - count
```

## Inter-VLAN traffic that should have been denied
Answers: is the firewall policy actually holding?

```spl
index=pfsense action=block
| stats count by src_ip, dest_ip, dest_port
| sort - count
```

## New outbound destinations from the IoT VLAN
Answers: has an IoT device started talking to something it did not talk to before?

```spl
index=suricata event_type=flow src_ip=<iot-vlan-cidr>
| stats earliest(_time) as first_seen count by dest_ip
| where first_seen > relative_time(now(), "-7d@d")
```

## Dashboards
(record dashboard definitions here as they are built)

---
date: '2026-03-07T13:21:11Z'
draft: true
title: 'Test'
tags: []
---

{{< mermaid >}}
---
title: "test"
---
flowchart TD
    Start --> Stop
{{< /mermaid >}}

{{< mermaid >}}
---
title: "CoAP Message"
---
packet
0-1: "Version"
2-3: "Type"
4-7: "Token length"
8-15: "Code"
16-31: "Message ID"
32-63: "Token (Variable Length)"
64-95: "Options (Multiple, Variable Length)"
96-103: "11111111"
104-127: "Payload (Variable Length)"
{{< /mermaid >}}
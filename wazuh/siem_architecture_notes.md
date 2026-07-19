Snort on its own only detects and logs, it has no memory of past alerts and no way to compare itself against other log sources. The three things Wazuh adds are: 
- Centralized collection, so alerts don't need someone physically watching a log file
- Normalization of different log formats into a common structure so their fields can be compared
- Correlating a persistent knowledge base of past events that new events get checked against within a time window, so related activity can be flagged as connected 

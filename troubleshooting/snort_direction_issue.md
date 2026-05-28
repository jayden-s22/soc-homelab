Issue: Rule with -> wasn't executing
Cause: 530 response from FTP travels from server to client, not client to server. Rule was matching in the wrong direction.
Solution: Rewrote with explicit source ($HOME_NET 21) and flow:established,to_client keyword.
Learned: Snort is largely stateless, so directionality must be explicitly defined.

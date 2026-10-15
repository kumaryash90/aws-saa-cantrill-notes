# Other Services and Features

## AWS Local Zones

- because distance causes latency, hence need for infra closer to premises
- there can be multiple local zones in a city, connected to a parent region (but operating independently of each other)
- also supports Direct Connect
- local zones have private networking with the parent region
- EBS snapshots use S3 in the parent region
- 1 zone → no built-in resilience
- like an AZ, but near your location

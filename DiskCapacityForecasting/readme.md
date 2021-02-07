# DiskCapacityForecasting

There are two versions provided for this dashboard:

`disk_analysis_sai.xml`:  (metric index data) For environments that already use Splunk App for Infrastructure. Supports environments that contain both Windows and *Nix hosts. This dashboard expects the SAI macro `sai_metrics_indexes` to exist.

`disk_analysis_scom.xml`: (event index data) For environments that have SCOM/perfmon data in a standard "event" index. Only supports windows hosts.

With one or the other of these dashboards it should hopefully be easy enough to convert this dashboard to fit other common data sources.

Hit me up via a Github issue if you get stuck porting these dashboards.


# Interogarea unui cluster

Deschide Kibana pe http://localhost:5601. În **Dev Tools** pentru a interoga sănătatea unui cluster vei interoga pe API cu `GET /_cluster/health`. Vei primi un răspuns similar cu următorul obiect JSON:

```json
{
  "cluster_name" : "aplicatii",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 63,
  "active_shards" : 63,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 38,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 62.37623762376238
}
```

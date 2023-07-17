# gitlab-datadog-K8-minikube

## What is this?

A lab based repo that deploys a Gitlab instance with Prometheus enabled, wit a Datadog agent sidecar to send those metrics to Datadog

## How does it work? 

### Deploy Gitlab

1. Add the Helm repos for Gitlab and Datadog
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo add gitlab https://charts.gitlab.io/
helm repo update
```

2. Start the minikube cluster with bumped up resources (Gitlab is hungry for RAM and CPU)

```bash
minikube start \
  --driver=virtualbox \
  --cpus 4 \
  --memory 8192
```

3. Enable the required plugins

```bash
minikube addons enable ingress
minikube addons enable dashboard
```

4. Check minikube is working ok

```shell
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

5. Start the Gitlab pod

```bash
helm upgrade --install gitlab gitlab/gitlab \
   --timeout 600s   \
   --set global.hosts.domain=$(minikube ip).nip.io \
   --set global.hosts.externalIP=$(minikube ip) \
   -f gitlab-values.yaml
```

6. Wait for Gitlab to deploy

```bash
kubectl get pods -w
```

** Note: This can take 10-15 minutes, and seems to error out on some pods, be patient and it'll generally work itself out **

7. Check Gitlab accesable in the browser using the Minikube IP

```bash
open https://gitlab.$(minikube ip).nip.io/
```

8. Get the root password and login to the frotend with the username `root`

```
kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
```

9. Change the annotations for the Gitlab URL to the new minikube URL:

Before:
```
"gitlab_url":"https://gitlab.changeme.nip.io/",
```

```
$ minikube ip
192.168.59.101


After:

```
echo minikube ip
"gitlab_url":"https://gitlab.192.168.59.101.nip.io/",
```

### Deploy the Datadog Agent

1. Get an API key from your Datadog org

2. Start the Datadog agent

```bash
helm upgrade --install ddagent \
  -f ddagent-values.yaml \
  --set datadog.apiKey='ABC123' \
  datadog/datadog
```

### Check Gitlab Metrics are being sent to Datadog

1. Jump into the Datadog agent
```
kubectl exec -it ddagent-datadog-ztn78 -- /bin/bash
```

2. Run the Gitlab agent check
```
agent check gitlab
```

3. Check the results

```bash
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/check.go:309 in Configure) | python check configure done gitlab
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/loader.go:216 in Load) | python loader: done loading check gitlab (version 6.1.0)
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/scheduler.go:182 in getChecks) | Python Check Loader: successfully loaded check 'gitlab'
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/check.go:83 in runCheck) | Running python check gitlab (version: '6.1.0', id: 'gitlab:2212ac4945715abc')
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:245) | Starting new HTTP connection (1): 10.244.0.30:3807
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:473) | http://10.244.0.30:3807 "GET /metrics HTTP/1.1" 200 66333
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `consistency_checks` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `deployments` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `exporter_http_request_duration_seconds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `exporter_http_requests_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_maintenance_mode` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_memwd_max_memory_limit` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_rails_boot_time_seconds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_global_search_indexing_apdex_success_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_global_search_indexing_apdex_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_loose_foreign_key_clean_ups_apdex_success_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_loose_foreign_key_clean_ups_apdex_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_loose_foreign_key_clean_ups_error_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sli_loose_foreign_key_clean_ups_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_sql_primary_duration_seconds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_db_cached_count_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_db_count_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_db_primary_cached_count_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_db_primary_count_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_db_write_count_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `gitlab_transaction_event_stuck_jira_import_jobs_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `global_search_bulk_cron_initial_queue_size` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_gc_stat_compact_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_gc_stat_ext_heap_fragmentation` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_gc_stat_read_barrier_faults` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_gc_stat_total_moved_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_process_resident_anon_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `ruby_process_resident_file_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_enqueued_jobs_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_file_descriptors` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_duration_seconds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_compact_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_ext_heap_fragmentation` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_allocatable_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_allocated_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_available_slots` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_eden_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_final_slots` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_free_slots` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_live_slots` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_marked_slots` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_sorted_length` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_heap_tomb_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_major_gc_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_malloc_increase_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_malloc_increase_bytes_limit` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_minor_gc_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_old_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_old_objects_limit` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_oldmalloc_increase_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_oldmalloc_increase_bytes_limit` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_read_barrier_faults` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_remembered_wb_unprotected_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_remembered_wb_unprotected_objects_limit` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_total_allocated_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_total_allocated_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_total_freed_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_total_freed_pages` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_gc_stat_total_moved_objects` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_cpu_seconds_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_max_fds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_proportional_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_resident_anon_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_resident_file_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_resident_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_start_time_seconds` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_process_unique_memory_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_exporter_ruby_sampler_duration_seconds_total` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:58 UTC | CORE | WARN | (pkg/autodiscovery/providers/clusterchecks.go:197 in heartbeatSender) | Unable to send extra heartbeat to Cluster Agent, err: "https://10.105.80.63:5005/api/v1/clusterchecks/status/minikube" is unavailable: 502 Bad Gateway
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_load_balancing_count` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (mixins.py:821) | Skipping metric `sidekiq_mem_total_bytes` as it is not defined in the metrics mapper, has no transformer function, nor does it match any wildcards.
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:151) | checking readiness against https://gitlab.192.168.59.101.nip.io//-/readiness
2023-07-17 15:43:59 UTC | CORE | WARN | (pkg/collector/python/datadog_agent.go:131 in LogMessage) | gitlab:2212ac4945715abc | (http.py:389) | An unverified HTTPS request is being made to https://gitlab.192.168.59.101.nip.io//-/readiness
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:1014) | Starting new HTTPS connection (1): gitlab.192.168.59.101.nip.io:443
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:473) | https://gitlab.192.168.59.101.nip.io:443 "GET //-/readiness HTTP/1.1" 200 48
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:184) | gitlab check readiness succeeded
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:151) | checking liveness against https://gitlab.192.168.59.101.nip.io//-/liveness
2023-07-17 15:43:59 UTC | CORE | WARN | (pkg/collector/python/datadog_agent.go:131 in LogMessage) | gitlab:2212ac4945715abc | (http.py:389) | An unverified HTTPS request is being made to https://gitlab.192.168.59.101.nip.io//-/liveness
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:1014) | Starting new HTTPS connection (1): gitlab.192.168.59.101.nip.io:443
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:473) | https://gitlab.192.168.59.101.nip.io:443 "GET //-/liveness HTTP/1.1" 200 15
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:184) | gitlab check liveness succeeded
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:151) | checking health against https://gitlab.192.168.59.101.nip.io//-/health
2023-07-17 15:43:59 UTC | CORE | WARN | (pkg/collector/python/datadog_agent.go:131 in LogMessage) | gitlab:2212ac4945715abc | (http.py:389) | An unverified HTTPS request is being made to https://gitlab.192.168.59.101.nip.io//-/health
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:1014) | Starting new HTTPS connection (1): gitlab.192.168.59.101.nip.io:443
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:473) | https://gitlab.192.168.59.101.nip.io:443 "GET //-/health HTTP/1.1" 302 116
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | - | (connectionpool.py:473) | https://gitlab.192.168.59.101.nip.io:443 "GET /users/sign_in HTTP/1.1" 200 29420
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (gitlab.py:184) | gitlab check health succeeded
2023-07-17 15:43:59 UTC | CORE | DEBUG | (pkg/collector/python/datadog_agent.go:135 in LogMessage) | gitlab:2212ac4945715abc | (common.py:12) | GitLab token not found; please add one in your config to enable version metadata collection.
=== Series ===
[
  {
    "metric": "gitlab.ruby.gc_stat.remembered_wb_unprotected_objects",
    "points": [
      [
        1689608639,
        37806
      ]
    ],
```

The important part should be at the end where it lists the amount of metrics found:

```bash
=========
Collector
=========

  Running Checks
  ==============

    gitlab (6.1.0)
    --------------
      Instance ID: gitlab:2212ac4945715abc [OK]
      Configuration Source: container:docker://062d1eb38c93369deaf8c61fd1d0fe0d7e56e054703e1f6ef241638d10c58797
      Total Runs: 1
      Metric Samples: Last Run: 6,352, Total: 6,352
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 4, Total: 4
      Average Execution Time : 1.324s
      Last Execution Date : 2023-07-17 15:43:59 UTC (1689608639000)
      Last Successful Execution Date : 2023-07-17 15:43:59 UTC (1689608639000)
```

4. Check the results are showing up in the Datadog platform
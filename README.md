# fluent-bit-installation

*You can install fluent bit by running following command*

1. Create namespace logging 

kubectl create namespace logging

2. Deploy the fluent bit 

(**)
kubectl apply -f service-account.yaml
kubectl apply -f role.yaml
kubectl apply -f role-binding.yaml
kubectl apply -f configmap.yaml
kubectl apply -f ds.yaml
(**)

You can customize you configmap, In above used configmap we only fetching the logs from nginx namespace. If you want to fetch the logs from all the namespace change the config map Input and Output paramaeter as shown below.

(**)
  input-kubernetes.conf: |
    [INPUT]
        Name              tail
        Tag               kube.*
        Path              /var/log/containers/*.log
        Parser            docker
        DB                /var/log/flb_kube.db
        Mem_Buf_Limit     5MB
        Skip_Long_Lines   On
        Refresh_Interval  10

  output-elasticsearch.conf: |
    [OUTPUT]
        Name            es
        Match           *
        Host            ${FLUENT_ELASTICSEARCH_HOST}
        Port            ${FLUENT_ELASTICSEARCH_PORT}
        Logstash_Format On
        Replace_Dots    On
        Retry_Limit     False

(**)

Here we are using wildcard to fetch logs from all the namespaces.

If you want logs from all the namespaces except a few namespaces and you want to exclude those few namespaces. For that, you need to add an Exclude_path option in the Fluent-Bit configuration file. 

One can pass multiple namespaces names separated by a comma. For example, as shown below, We can exclude logs of multiple namespaces by giving their path separated by a comma in the Fluent-Bit configuration file (this way basically we can selectively exclude namespaces whose logs we don't want fluent-bit to collect and pass them).


(**) [INPUT]
         Name tail
         Path /var/log/containers/*.log
         Exclude_Path /var/log/containers/*_kube-system_*.log,/var/log/containers/*_default_*.log
         multiline.parser docker, cri
         Tag kube.*
         Mem_Buf_Limit 5MB
         Skip_Long_Lines On
(**)

Here, in the above example we are ignoring the kube-system and default namespace. You can add your namespace name in the same way and it will get ignored.

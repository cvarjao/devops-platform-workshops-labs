# My personal notes message

oc get all,template,secret -l template=postgresql-ephemeral-template


oc delete all,configmap,secret,pvc -l app=cvarjao-grafana
oc -n 4zq6uj-cvarjao-ocp201-tst-dev delete all,configmap,secret,pvc -l app=loki
oc -n 4zq6uj-cvarjao-ocp201-tst-dev delete all,configmap,secret,pvc -l app=prometheus
oc -n 4zq6uj-cvarjao-ocp201-tst-dev delete all,configmap,secret,pvc -l app=cvarjao-prometheus


oc expose service [service name] --name prometheus -l app=prometheus




oc apply -f cvarjao-prometheus.yaml
oc apply -f loki_template.yaml

oc expose service "$(oc get service -l app=loki -o custom-columns=NAME:.metadata.name --no-headers)"


oc new-app grafana/grafana:6.2.0 --name cvarjao-grafana
oc get all -l app=cvarjao-grafana
oc expose service "$(oc get service -l app=cvarjao-grafana -o custom-columns=NAME:.metadata.name --no-headers)"

oc rollout pause dc/cvarjao-grafana

oc create configmap cvarjao-grafana-dashboard-providers --from-file=dashboards.yaml=grafana-dashboard-providers.yaml 
oc label configmap/cvarjao-grafana-dashboard-providers app=cvarjao-grafana
oc set volumes dc/cvarjao-grafana --add -m /etc/grafana/provisioning/dashboards/ --configmap-name=cvarjao-grafana-dashboard-providers


oc create configmap cvarjao-grafana-datasources --from-file=datasources.yml=grafana-datasources.yaml 
oc label configmap/cvarjao-grafana-datasources app=cvarjao-grafana
oc set volumes dc/cvarjao-grafana --add -m /etc/grafana/provisioning/datasources --configmap-name=cvarjao-grafana-datasources

oc create configmap cvarjao-grafana-dashboards --from-file=simple_dashboard.json=simple_dashboard.json
oc label configmap/cvarjao-grafana-dashboards app=cvarjao-grafana
oc set volumes dc/cvarjao-grafana --add -m /var/lib/grafana/dashboards/ --configmap-name=cvarjao-grafana-dashboards

oc rollout resume dc/cvarjao-grafana

oc get is,svc,dc,route,configmap -l app=cvarjao-grafana -o yaml > grafana-template-source.yaml

oc get is,svc,dc,route,configmap -l app=cvarjao-grafana -o json > grafana-template-source.json

cat grafana-template-source.json | jq 'del(.items[].status) | del(.items[].metadata.uid) | del(.items[].metadata.selfLink) | del(.items[].metadata.resourceVersion) | del(.items[].metadata.namespace) | del(.items[].metadata.generation) | del(.items[].metadata.creationTimestamp) | .objects = .items | del(.items)' 


npx oc-clean-template-things --file=templates/grafana.yaml
npx oc-clean-template-things --file=templates/loki.yaml
npx oc-clean-template-things --file=templates/prometheus.yaml

oc -n 4zq6uj-cvarjao-ocp201-tst-dev process -f loki.yaml \
    -p NAME=cvarjao-loki \
    | oc -n 4zq6uj-cvarjao-ocp201-tst-dev create -f -

oc -n 4zq6uj-cvarjao-ocp201-tst-dev process -f prometheus.yaml \
    -p NAME=cvarjao-prometheus \
    -p NAMESPACE=4zq6uj-cvarjao-ocp201-tst-dev \
    -p ROUTE_HOST=cvarjao-prometheus-4zq6uj-cvarjao-ocp201-tst-dev \
    -p ROUTE_DOMAIN=pathfinder.gov.bc.ca \
    | oc -n 4zq6uj-cvarjao-ocp201-tst-dev create -f -

oc -n 4zq6uj-cvarjao-ocp201-tst-dev process -f grafana.yaml \
    -p GRAFANA_SERVICE_NAME=cvarjao-grafana \
    -p LOKI_SERVICE_NAME=cvarjao-loki \
    -p PROMETHEUS_SERVICE_NAME=cvarjao-prometheus \
    -p ROUTE_HOST=cvarjao-grafana-4zq6uj-cvarjao-ocp201-tst-dev \
    -p ROUTE_DOMAIN=pathfinder.gov.bc.ca \
    | oc -n 4zq6uj-cvarjao-ocp201-tst-dev create -f -

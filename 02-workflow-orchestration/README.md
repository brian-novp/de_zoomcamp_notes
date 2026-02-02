# What is Kestra?
- Workflow orchestrator that is language-agnostic.
- It uses plugins to orchestrate workflow.
- It has no code and AI support.
- It has web app like Airflow. We can edit the yaml file (Flow, Kestra call it) inside the web app. It also has built-in documentation. If we click the specific task, the documentation will automatically display the corresponding plugin documentation.

# Metadata storage
- Kestra uses PostgreSQL as metadata storage. Same as Airflow. (must be included in docker compose yaml for local development)

# Then what is/are the difference(s) with Airflow?
- Kestra adopts Infrastructure as Code (IaC) practices for data and process orchestration, using a declarative language within its YAML files. 
- Airflow uses Python to orchestrate workflow (as of 2026).
- Airflow uses XCOM backend to communicate data between tasks or functions. It can use Postgre(Max of 1GB), SQLite(2GB max), MySQL (64kb). Airflow can also use custom backedn like object storage (AWS S3), if we need to pass data between task that exceed the ability of the metadata database.
- By default, Kestra uses internal storage, which is the equivalent of XCOM in Airflow. It saves files generated during a flow execution and passes them between tasks via `outputs`. [Kestra Internal Storage Docs](https://kestra.io/docs/architecture/main-components#internal-storage)  
- Airflow has role based access control without paying. While we must pay the Kestra enterprise edition to enable RBAC.
- Airflow has some ways to store secrets (using variables or in connections setting) without paying [Airflow Secret](https://airflow.apache.org/docs/apache-airflow/stable/security/secrets/index.html).  While we must pay the Kestra enterprise edition to enable storing secrets.  

# Breaking Down Kestra yaml syntax
## id and namespace
```yaml
id : getting_started
namespace : zoomcamp
```  
`id` will be displayed in the Flow menu. `namespace` is used as a workspace group. If we store key value or secret in the same namespace, then we can use it anywhere in the tasks or flows without creating new id or variables. Pretty convenient feature, so, make sure these two are correctly setup. Once we saved the yaml, there is no way to change id and namespace but to remove the existing yaml and create new one.  

## variables
```yaml
variables:
  data : green_taxi.csv
  table: public.green_taxi_trip
```
accessing variables described under `variables` can be done in two ways:
1. Call it in a peeble expression directly {{ vars.data }} will display green_taxi.csv  
2. when encountering:
```yaml
variables:
  file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
  staging_table: "public.{{inputs.taxi}}_tripdata_staging"
  table: "public.{{inputs.taxi}}_tripdata"
  data: "{{outputs.extract.outputFiles[inputs.taxi ~ '_tripdata_' ~ inputs.year ~ '-' ~ inputs.month ~ '.csv']}}"
```  
It is important to add `{{render(vars."the nameof the variable that we want to use")}}`.  `render` will recursively translate all of the peeble templating and output the final result rather than just output the `{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv`  

For example, in a task of `set_label` we want to access the file variable, so we must do it like so
```yaml
tasks:
  - id: set_label
    type: io.kestra.plugin.core.execution.Labels
    labels:
      file: "{{render(vars.file)}}"
      taxi: "{{inputs.taxi}}"
```  
notice that `file` variable has `render` in it before the `vars.file`. See [kestra expression documentation](https://kestra.io/docs/expressions)

## task
Task is the core of Kestra. It is plugin-based. In it, we can use many properties. These properties are configurable. For example  
```yaml
tasks:
- id: extract
    type: io.kestra.plugin.scripts.shell.Commands
    outputFiles:
      - "*.csv"
    taskRunner:
      type: io.kestra.plugin.core.runner.Process
    commands:
      - wget -qO- https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{{inputs.taxi}}/{{render(vars.file)}}.gz | gunzip > {{render(vars.file)}}
```  
- `id` is the identity that will be displayed in execution logs and gantt charts
- `type` is where we put the plugins for the task. In this is case, it tells kestra that in "id extract" need to run command line
- `ouputFiles` is a property of the `io.kestra.plugin.scripts.shell.Commands`, it tells Kestra that the output from the command line will be saved as csv
-  `taskRunner` is another property of `io.kestra.plugin.scripts.shell.Commands`  [shell command](https://kestra.io/plugins/plugin-script-shell/io.kestra.plugin.scripts.shell.commands)
- `type` another type where we put the plugin of task runner that executes a task as a subprocess on the Kestra host. [runner process](https://kestra.io/plugins/core/runner/io.kestra.plugin.core.runner.process)
- `commands` contains the shell command to run (it is a property of `io.kestra.plugin.scripts.shell.Commands` plugin)

## pluginDefaults
```yaml
pluginDefaults:
  - type: io.kestra.plugin.gcp
    values:
      serviceAccount: "{{kv('GCP_CREDS')}}"
      projectId: "{{kv('GCP_PROJECT_ID')}}"
      location: "{{kv('GCP_LOCATION')}}"
      bucket: "{{kv('GCP_BUCKET_NAME')}}"
```  
to setup cloud services. for `serviceAccount` in DE zoomcamp uses key value store. This is because SECRET feature in Kestra only available in Enterprise edition. Best practice is to use SECRET feature. To access the value of a key, use `kv` before accesing the key "{{kv('GCP_BUCKET_NAME')}}
in this case we use Google Cloud Platform plugin to setup the BigQuery and Google Cloud Storage. [Kestra pluginDefaults](https://kestra.io/docs/workflow-components/plugin-defaults)  


## trigger
```yaml
triggers:
  - id: green_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 1 * *"
    inputs:
      taxi: green
```  
Use triggers to setup well...triggers! In this case we use Schedule trigger using a cron expression: to run this flow at 9 AM every start of the month when the flow detecs taxi input is green. Kestra triggers come with many flavours. [Triggers](https://kestra.io/docs/workflow-components/triggers)
# What is Kestra?
- Workflow orchestrator that is language-agnostic.
- It has no code and AI support.
- It has web app like Airflow.

# Metadata storage
- Kestra uses PostgreSQL as metadata storage. Same as Airflow. (must be included in docker compose yaml)

# Then what is/are the difference(s) with Airflow?
- Kestra adopt IaC practices for data and process orchestration.
- Therefore Kestra uses declarative language in yaml file
- Airflow use Python to orchestrate workflow (as of 2026)

# Breaking Down Kestra yaml syntax
## id and namespace  
```yaml
id : getting_started
namespace : zoomcamp
```
    once we saved, there is no way to change id and namespace but to remove the existing yaml and create new one.  
    id 

# Important things (to me)
when encountering:
```yaml
variables:
  file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
  staging_table: "public.{{inputs.taxi}}_tripdata_staging"
  table: "public.{{inputs.taxi}}_tripdata"
  data: "{{outputs.extract.outputFiles[inputs.taxi ~ '_tripdata_' ~ inputs.year ~ '-' ~ inputs.month ~ '.csv']}}"
```  
It is important to add `{{render(vars."the nameof the variable that we want to use")}}`.  `render` will recursively translate all of the jinja templating and output the final result rather than just output the `{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv`
For example, in a task of `set_label` we want to access the file variable, so we must do it like so
```yaml
tasks:
  - id: set_label
    type: io.kestra.plugin.core.execution.Labels
    labels:
      file: "{{render(vars.file)}}"
      taxi: "{{inputs.taxi}}"
```



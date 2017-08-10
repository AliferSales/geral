# Introduction
Unfortunately, the process of removing a DAG is not as simple as its creation. Just removing the file (_or the lines of code_) from the `dags/` folder does not remove it from the web GUI.

This tutorial will teach you why this happens and how to get around it.

If you are not interested in the step by step process, find at the end of this tutorial a function that automates the removal of DAG.

# Why doesn't is sufficient to delete the DAG file?
When Airflow recognizes a new DAG through a file in the `dags /` folder, it takes care of inserting it into its database. The problem is that when the lines of the DAG are deleted, the records that were made in the database are not and so the dag remains present in the web GUI.

# Solution
That way, to remove the DAG, simply remove all your records from the database. It is considered from here that the database used is the postgree.

For example, suppose you want to remove the dag identified by `dag_test`.

> **Observation**
> Be sure to refer to dag correctly using `dag_id`. It is the name of the name that appears in the GUI web.

![001](https://github.com/CTS-FGV/geral/blob/master/tutoriais/img/remover_dag_001.png).

1. Na pasta `airflow/`, abra o banco de dados.

    ```
    $ sqlite3 airflow.db
    ```

2. Remova os registros das tabelas rodando os códigos abaixo.

    ```
    delete from airflow.xcom where dag_id = 'dag_test';
    delete from airflow.task_instance where dag_id = 'dag_test';
    delete from airflow.sla_miss where dag_id = 'dag_test';
    delete from airflow.log where dag_id = 'dag_test';
    delete from airflow.job where dag_id = 'dag_test';
    delete from airflow.dag_run where dag_id = 'dag_test';
    delete from airflow.dag where dag_id = 'dag_test';
    ```
    
DONE! DAG removida da web GUI :smile: 

# Função geral de remoção

Crie o arquivo `delete_dag` com o conteúdo abaixo:

```
import sqlite3
import sys

conn = sqlite3.connect('airflow.db')
c = conn.cursor()

dag_input = sys.argv[1]

for t in ["xcom", "task_instance", "sla_miss", "log", "job", "dag_run", "dag" ]:
    query = "delete from {} where dag_id='{}'".format(t, dag_input)
    c.execute(query)

conn.commit()
conn.close()
```

agora, basta chamar o arquivo, indicando a dag_id da DAG que se deseja remover.

```
$ python3 delete_dag.py dag_id
```

---
refer: https://stackoverflow.com/a/44530631/7432019
writed by: Alifer Sales

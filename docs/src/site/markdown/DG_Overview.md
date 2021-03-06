

[::Go back to Oozie Documentation Index::](index.html)

# Oozie Workflow Overview

Oozie is a server based _Workflow Engine_ specialized in running workflow jobs with actions that run Hadoop Map/Reduce
and Pig jobs.

Oozie is a Java Web-Application that runs in a Java servlet-container.

For the purposes of Oozie, a workflow is a collection of actions (i.e. Hadoop Map/Reduce jobs, Pig jobs) arranged in
a control dependency DAG (Directed Acyclic Graph). "control dependency" from one action to another means that the second
action can't run until the first action has completed.

Oozie workflows definitions are written in hPDL (a XML Process Definition Language similar to
[JBOSS JBPM](http://www.jboss.org/jbossjbpm/) jPDL).

Oozie workflow actions start jobs in remote systems (i.e. Hadoop, Pig). Upon action completion, the remote systems
callback Oozie to notify the action completion, at this point Oozie proceeds to the next action in the workflow.

Oozie uses a custom SecurityManager inside its launcher to catch exit() calls from the user code. Make sure to delegate checkExit()
calls to Oozie's SecurityManager if the user code uses its own SecurityManager. The Launcher also grants java.security.AllPermission
by default to the user code.

Oozie workflows contain control flow nodes and action nodes.

Control flow nodes define the beginning and the end of a workflow ( `start`, `end` and `fail` nodes) and provide a
mechanism to control the workflow execution path ( `decision`, `fork` and `join` nodes).

Action nodes are the mechanism by which a workflow triggers the execution of a computation/processing task. Oozie
provides support for different types of actions: Hadoop map-reduce, Hadoop file system, Pig, SSH, HTTP, eMail and
Oozie sub-workflow. Oozie can be extended to support additional type of actions.

Oozie workflows can be parameterized (using variables like `${inputDir}` within the workflow definition). When
submitting a workflow job values for the parameters must be provided. If properly parameterized (i.e. using different
output directories) several identical workflow jobs can concurrently.

## WordCount Workflow Example

**Workflow Diagram:**

<img src="./DG_Overview.png"/>

**hPDL Workflow Definition:**


```
<workflow-app name='wordcount-wf' xmlns="uri:oozie:workflow:0.1">
    <start to='wordcount'/>
    <action name='wordcount'>
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.myorg.WordCount.Map</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.myorg.WordCount.Reduce</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>${inputDir}</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>${outputDir}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to='end'/>
        <error to='end'/>
    </action>
    <kill name='kill'>
        <message>Something went wrong: ${wf:errorCode('wordcount')}</message>
    </kill/>
    <end name='end'/>
</workflow-app>
```

[::Go back to Oozie Documentation Index::](index.html)



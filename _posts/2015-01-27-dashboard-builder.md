---
layout: blog
title:  "Debug dashboard-builder"
date:   2015-01-27 18:40:12
categories: jboss
permalink: /dashboard-builder
author: Kylin Soong
duoshuoid: ksoong20150127
---

This article show some viewpoints for how [dashboard-builder](https://github.com/droolsjbpm/dashboard-builder/tree/master/modules) works.

## Deploy dashboard-builder to WildFly

* Clone the source code via `git clone git@github.com:droolsjbpm/dashboard-builder.git`
* Build modules via `./build.sh h2`
* Execute Maven command `mvn clean install -Dfull -DskipTests` under `builder` directory will generate `dashbuilder-6.3.0-SNAPSHOT-wildfly8.war`
* Deploy `dashbuilder-6.3.0-SNAPSHOT-wildfly8.war` to WildFly.

## Use Mysql with dashboard-builder

* [Step by step procedure](https://github.com/droolsjbpm/dashboard-builder/blob/master/builder/src/main/wildfly8/README.md)
* [Set up Datasource](https://github.com/jbosschina/wildfly-dev-cookbook/blob/master/persistence/create-ds-mysql.cli)

## ControllerServlet init

ControllerServlet is the entry point for UI request, all request as below will go to ControllerServlet, this section dive into ControllerServlet init

* /workspace/*
* /jsp/*
* /kpi/*

**init App Directories** 

The following directory fields be added to `org.jboss.dashboard.Application`:

* tmp/vfs/temp/temp7e145d2fd5273f6a/dashbuilder-6.3.0-SNAPSHOT-wildfly8.war-c7ba2b28938c64da
* tmp/vfs/temp/temp7e145d2fd5273f6a/dashbuilder-6.3.0-SNAPSHOT-wildfly8.war-c7ba2b28938c64da/WEB-INF/etc
* tmp/vfs/temp/temp7e145d2fd5273f6a/dashbuilder-6.3.0-SNAPSHOT-wildfly8.war-c7ba2b28938c64da/WEB-INF/lib

**Startable Start**

* `org.jboss.dashboard.database.hibernate.HibernateInitializer`
* `org.jboss.dashboard.cluster.ClusterNodesManager`
* `org.jboss.dashboard.workspace.SkinsManagerImpl`
* `org.jboss.dashboard.workspace.PanelsProvidersManagerImpl`
* `org.jboss.dashboard.workspace.EnvelopesManagerImpl`
* `org.jboss.dashboard.workspace.LayoutsManagerImpl`
* `org.jboss.dashboard.ui.resources.ResourceManagerImpl`
* `org.jboss.dashboard.security.UIPolicy`
* `org.jboss.dashboard.DeploymentScanner`
* `org.jboss.dashboard.initialModule.InitialModulesManager`

**Hibernate cfg**

~~~
org/jboss/dashboard/ui/resources/GraphicElement.hbm.xml
org/jboss/dashboard/workspace/Workspace.hbm.xml
org/jboss/dashboard/workspace/PanelParameter.hbm.xml
org/jboss/dashboard/workspace/Panel.hbm.xml
org/jboss/dashboard/workspace/PanelInstance.hbm.xml
org/jboss/dashboard/workspace/Section.hbm.xml
org/jboss/dashboard/database/InstalledModule.hbm.xml
org/jboss/dashboard/database/DataSourceEntry.hbm.xml
org/jboss/dashboard/cluster/ClusterNode.hbm.xml
org/jboss/dashboard/security/PermissionDescriptor.hbm.xml
org/jboss/dashboard/ui/panel/advancedHTML/HtmlCode.hbm.xml
org/jboss/dashboard/ui/panel/dataSourceManagement/DataSourceTableEntry.hbm.xml
org/jboss/dashboard/ui/panel/dataSourceManagement/DataSourceColumnEntry.hbm.xml
org/jboss/dashboard/provider/DataProvider.hbm.xml
org/jboss/dashboard/kpi/KPI.hbm.xml
~~~

## HibernateInitializer

[org.jboss.teiid.dashboard.hibernate.HibernateInitializerTest](https://github.com/kylinsoong/teiid-samples/blob/master/dashboard/src/main/java/org/jboss/teiid/dashboard/hibernate/HibernateInitializerTest.java)



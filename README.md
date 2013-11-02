      
# What is this?
 
This CM2.0 template shows how to customize CM2.0 for use in production. It is based on the WAR overlay UMD is running in production.
All references to UMD have been removed and replaced with 'myuni' (short for my university).  You can customize this template by doing 
a full text search and replacing all references to 'myuni' with your university (more info on customization below).

# TODO
 
- Clean up the config files and add comments above each line explaining what it does (maybe add a link to the docs)

# Features
 
- Shows how to specify config file location in command line so you can run multiple versions of CM w/ different config files

# Before you start

- For training developer installs you are *strongly urged* to keep the database users names and all directory paths the same
  as are specified in this document (with the exception of changing all references to C:/Users/cmann/ to your home directory)  


# Installation

## Install pre-reqs

- Oracle XE
- Git http://git-scm.com/downloads
- Maven: http://maven.apache.org/



## Install the config files

- Copy /training/maven/settings.xml to C:/Users/cmann/.m2  (create '.m2' directory in your home directory - replace cmann)

- Copy all files in /training/config/ into C:/Users/cmann/kuali/main/myuni20/      (replace Users/cmann with your home directory)


# Startup

- The commands below should be run from the DOS prompt or shell prompt 

## Checkout, build, and start Rice

### Checkout CM version of Rice (w/post processor etc)

```
mkdir /ks-201
cd /ks-201
svn co https://svn.kuali.org/repos/student/enrollment/aggregate/tags/student-2.0.1-cm/
```

### Load CM database

``` 
cd /ks-201/student-2.0.1-cm/ks-deployments/ks-cfg-dbs/ks-app-db
mvn clean install -Pks-db,oracle -Dumd.cm.additional.config.locations=C:/Users/cmann/kuali/main/myuni20/myuni-cm-config-20.xml -Dks.impex.url=jdbc:oracle:thin:@localhost:1521:XE -Dks.impex.dba.password=sys  -Dks.impex.dba.username="sys as sysdba" -Dks.impex.username=CMMYUNI -Dks.impex.password=CMMYUNI
```
- See https://wiki.kuali.org/display/STUDENTDOC/%28CM+2.0%29+3.1.3+Load+the+Kuali+Student+Application+Data 

- Edit the mvn clean install command above to use whatever your "sys as sysdba" password is

### Load RICE database with student data 

see https://wiki.kuali.org/display/STUDENTDOC/%28CM+2.0%29+3.1.4+Load+the+Kuali+Rice+Data

```
cd /ks-201/student-2.0.1-cm/ks-deployments/ks-cfg-dbs/ks-rice-db
mvn clean install -Denforcer.skip=true -Pks-db,oracle -Dumd.cm.additional.config.locations=C:/Users/cmann/kuali/main/myuni20/myuni-cm-config-20.xml -Dks.impex.url=jdbc:oracle:thin:@localhost:1521:XE -Dks.impex.dba.password=sys  -Dks.impex.dba.username="sys as sysdba" -Dks.impex.username=RICEMYUNI -Dks.impex.password=RICEMYUNI
```

- Edit the mvn clean install command above to use whatever your "sys as sysdba" password is

### Start Rice 

```
cd /ks-201/student-2.0.1-cm/ks-deployments/ks-web/ks-rice-standalone
set MAVEN_OPTS=-Xmx1g -Xss8m -XX:MaxPermSize=512m -Xnoagent -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8003
mvn -Denforcer.skip=true -DskipTests -Pstandalone tomcat:run-war -Dadditional.config.locations=C:/Users/cmann/kuali/main/myuni20/myuni-rice-config-20.xml
```
Note: Setting the MAVEN_OPTS will start the tomcat Rice is running on in debug mode (listening on port 8003).  You can then connect to it
      using the eclipse debugger if you want to see what rice is doing (e.g. calling the post processors etc). 

-  Verify it is running on URL: http://localhost:8081/ks-rice-standalone-dev/
-  Login as 'admin' on the login screen.  If rice does not appear at that URL in your web browser, do not proceed further.  You need to fix rice first!
-  Make sure to leave Rice running in the background
-  Open a new DOS prompt to run the commands below


# Check-out, build, and start the CM WAR overlay


## Checkout CM Overlay Template

```
git remote add origin https://github.com/cmann50/myuni-cm-20.git
git clone https://github.com/cmann50/myuni-cm-20.git
```

## Start CM Overlay
	
Note: You can start to overlay from the command line to see it work.  You should import
it into Eclipse if you want to customize it or if you are in training

```
cd myuni-web/myuni-with-rice-embedded/
mvn clean tomcat:run-war -Dmyuni.cm.additional.config.locations=C:/Users/cmann/kuali/main/myuni20/myuni-cm-config-20.xml
```

Note: We changed additional.config.locations to myuni.cm.additional.config.locations so we could 
run rice and cm in same tomcat instance in DEV and QA (they used the same variable causing conflicts).

- Finally, use your web browser to verify CM is running on URL: http://localhost:8080/myuni/ 



# Customizing

- Start by running a full text search in eclipse to replace all matches of 'myuni' 
   with your university initials (e.g. UW, UMD, etc). Do two case sensitive search and then a final non-case
   sensitive search to get all matches.
   
- Change all directory names and files names to replace myuni with your university initials (e.g. UW, UMD, etc).


# Dump CM SQL

-  Uncomment the p6spy lines at the bottom of the my-cm-config-20.xml file if you want to dump all SQL to logs / console

# Dump Rice SQL

- Copy spy.properties to rice dir (it will log to file c:\temp\
```
cp C:\Users\cmann\git\ks-loader\ks-loader\training\debug\rice\spy.properties C:\ks-201\student-2.0.1-cm\ks-deployments\ks-web\ks-rice-standalone\src\main\resources
```

- Edit C:\ks-201\student-2.0.1-cm\ks-deployments\ks-web\ks-rice-standalone and add this at the bottom of the dependencies section:
```     
   <dependency>
       <groupId>p6spy</groupId>
       <artifactId>p6spy</artifactId>
       <version>1.3</version>
   </dependency>
 ```
- Uncomment P6 lines at the bottom of C:\Users\cmann\kuali\main\myuni20\myuni-rice-config-20.xml 

# Errors

## Unexpected EOF in prolog

```
	Caused by: com.ctc.wstx.exc.WstxEOFException: Unexpected EOF in prolog
 	  at [row,col {unknown-source}]: [1,0]
```
    
- Caused by bad encryption header.  Best way to troubleshoot is to put a breakpoint where the exception is (in the interceptor)
  and look at the variables.  You can see the data coming across.  Another way is to go directly to the service bus
  WSDL at http://localhost:8081/ks-rice-standalone-dev/services/soap/ksb/v2_0/serviceRegistry?wsdl   .  If it redirects to
  the login page, you've go the URL wrong.  Also, try turning off encryption.

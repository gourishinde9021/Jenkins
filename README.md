## Jenkins
Declarative Pipeline:
Enclosed in :

	  	pipeline {}

- The top-level of the Pipeline must be a block, specifically: pipeline { }.

- No semicolons as statement separators. Each statement has to be on its own line.

- Blocks must only consist of Sections, Directives, Steps, or assignment statements.

- A property reference statement is treated as a no-argument method invocation. So, for example, input is treated as input().

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Sections in pipeline:**

1. agent - mandatory
   Two ways to declare agents -
     1. Top Level Agents - declared at the top level of a Pipeline, an agent is allocated and then the timeout option is applied
     2. Stage Agents - declared within a stage, the options are invoked before allocating the agent and before checking any when conditions.
```
		pipeline {
			agent any -----> OR(none)  // Top Level Agents
			options {
				timeout(time: 1, unit: 'SECONDS')   // Timeout counter starts AFTER agent is allocated
			}
			stages {
				stage('Example') {
					agent any                		       // Stage Agents							
					options {
						timeout(time: 1, unit: 'SECONDS')  // Timeout counter starts BEFORE agent is allocated, This timeout will include the agent provisioning time
					}
					steps {
						echo 'Hello World'
					}
				}
			}
		}
```

**agent** - 
Different types of parameters
1. any - Execute the Pipeline, or stage, on any available agent. Ex: agent any
2. none - no global agent will be allocated for the entire Pipeline run and each stage section will need to contain its own agent section. Ex. agent none
3. label - an agent available in the Jenkins environment with the provided label. Ex. agent { label 'my-defined-label' }
4. node - works same as label, but node allows for additional options (such as customWorkspace). Ex. agent { node { label 'labelName' } }
5. docker - Execute the Pipeline, or stage, with the given container which will be dynamically provisioned on a node pre-configured to accept Docker-based Pipelines, or on a node matching the optionally defined label parameter.
Ex.
```
  agent {
      docker {
          image 'maven:3.9.3-eclipse-temurin-17'
          label 'my-defined-label'
          args  '-v /tmp:/tmp'
  		registryUrl 'https://myregistry.com/'
          registryCredentialsId 'myPredefinedCredentialsInJenkins'
  		}
  }
```
6. dockerfile - Execute the Pipeline, or stage, with a container built from a Dockerfile contained in the source repository. Ex : agent { dockerfile true }
```
	  agent {
	  			// Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
	  			dockerfile {
	  				filename 'Dockerfile.build'
	  				dir 'build'
	  				label 'my-defined-label'
	  				additionalBuildArgs  '--build-arg version=1.0.2'
	  				args '-v /tmp:/tmp'
	  				registryUrl 'https://myregistry.com/'
	  				registryCredentialsId 'myPredefinedCredentialsInJenkins'
	  	}
```
7. Kubernetes - Execute the Pipeline, or stage, inside a pod deployed on a Kubernetes cluster. The Pod template is defined inside the kubernetes { } block.
   
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

2. post - not mandatory

The post section defines one or more additional steps that are run upon the completion of a Pipeline’s or stage’s run (depending on the location of the post section within the Pipeline).
post can support any of the following post-condition blocks: always, changed, fixed, regression, aborted, failure, success, unstable, unsuccessful, and cleanup.
These condition blocks allow the execution of steps inside each condition depending on the completion status of the Pipeline or stage.

```
pipeline {
	agent any
	stages {
		stage ('Exmaple'){
			steps {
				echo "hello World"
			}
		}
	}
	post {
		always {
			echo "I will always say Hello again !!"
		}
	}
}
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

3. stages - Mandatory

Containing a sequence of one or more stage directives.

```
pipeline {
	agent any
	stages {                     //	The stages section will typically follow the directives such as agent, options etc.
		stage('Example') {
			steps {
				echo 'Hello World'
			}
		}
	}
}
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

4. steps - Mandatory   ( Allowed - Inside each stage block )

```
	stages {
		stage('Example') {
			steps {                   // The steps section must contain one or more steps.
				echo 'Hello World'
				}
			}
	}	
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Directives in Pipeline
----------------------

1. environment - specifies a sequence of key-value pairs which will be defined as environment variables for all steps, supports a special helper method credentials().

```  
	pipeline {
		agent any
		environment {
			CC = "clang"
		}
		stages {
			stage('Example Username/Password') {
				environment {
					SERVICE_CREDS = credentials('my-predefined-username-password')
				}
				steps {
					sh 'echo "Service user is $SERVICE_CREDS_USR"'
					sh 'echo "Service password is $SERVICE_CREDS_PSW"'
					sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
				}
			}
			stage('Example SSH Username with private key') {
				environment {
					SSH_CREDS = credentials('my-predefined-ssh-creds')
				}
				steps {
					sh 'echo "SSH private key is located at $SSH_CREDS"'
					sh 'echo "SSH user is $SSH_CREDS_USR"'
					sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
				}
			}
		}
	}
```	
	
2. options - Pipeline provides a number of these options, such as buildDiscarder, but they may also be provided by plugins, such as timestamps.

```
  options {
	timeout(time: 1, unit: 'HOURS') 
   }
```  

3. parameters - provides a list of parameters that a user should provide when triggering the Pipeline

```	
	parameters {
	        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
	        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
	        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
	        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
	        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
	}
	stages {
	    stage('Example') {
	        steps {
	            echo "Hello ${params.PERSON}"
	            echo "Biography: ${params.BIOGRAPHY}"
		}
		}
	}
```

4. triggers - automated ways in which the Pipeline should be re-triggered. cron, pollSCM and upstream
   Example:

```
		triggers { cron('H */4 * * 1-5') }
		triggers { pollSCM('H */4 * * 1-5') }
		triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }
```

5. jenkins cron syntax - Minutes, hours, Day Of month, Month, Day of Week

```
 	triggers{ cron('H/15 * * * *') }
```

6. stage - The stage directive goes in the stages section and should contain a steps section, an optional agent section, or other stage-specific directives.

7. tools - A section defining tools to auto-install and put on the PATH. This is ignored if agent none is specified.
	Supported Tools
		maven
		jdk
		gradle
	
	Example: 

```	
	pipeline {
		agent any
		tools {
			maven 'apache-maven-3.0.1'    // 	The tool name must be pre-configured in Jenkins under Manage Jenkins → Tools
	}
	stages {
		stage('Example') {
			steps {
				sh 'mvn --version'
				}
			}
		}
	}
```
	
10. input - The input directive on a stage allows you to prompt for input, using the input step. 

```
	 pipeline {
			agent any
			stages {
				stage('Example') {
					input {
						message "Should we continue?"
						ok "Yes, we should"
						submitter "alice,bob"
						parameters {
							string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
						}
					}
					steps {
						echo "Hello ${PERSON}, nice to meet you."
					}
				}
			}
		}
```

11. when - The when directive allows the Pipeline to determine whether the stage should be executed depending on the given condition.
		  Conditional structures can be built using the nesting conditions: not, allOf, or anyOf.

Built-in Conditions : 
1. branch - Execute the stage when the branch being built matches the branch pattern
	- EQUALS for a simple string comparison
			Example: when { branch 'master' }
			
	- GLOB (the default) for an ANT style path glob (same as for example changeset), for an ANT style path glob case insensitive (caseSensitive parameter)
	- For regular expression matching

```
			when { branch pattern: "release-\\d+", comparator: "REGEXP"}

			pipeline {
				agent any
				stages {
					stage('Example Build') {
						steps {
							echo 'Hello World'
						}
					}
					stage('Example deploy') {
						when {
							branch 'production'
							environment name: 'DEPLOY_TO', value: 'production'  // nested condition
							anyOf {                // multiple nested condition
								environment name: 'DEPLOY_TO', value: 'production'
								environment name: 'DEPLOY_TO', value: 'staging'
							}
						}
						steps {
							echo 'Deploying'
						}
					}
				}
			}
```
		
2. buildingTag - Execute the stage when the build is building a tag.
       Example:

```
	when { buildingTag() }
```
		
3. changelog - Execute the stage if the build’s SCM changelog contains a given regular expression pattern
       Example:
       
```
	when { changelog '.*^\\[DEPENDENCY\\] .+$' }
```

4. changeset - Execute the stage if the build’s SCM changeset contains one or more files matching the given pattern.
       Example:
       EQUALS -
       
```
	when { changeset "**/*.js" }
```

       REGEXP -
       
```
    	when { changeset pattern: ".TEST\\.java", comparator: "REGEXP" }
```

       GLOB -
       
```
    	when { changeset pattern: "*/*TEST.java", caseSensitive: true }
```

5. changeRequest - Executes the stage if the current build is for a "change request" (Pull Request), When no parameters are passed the stage runs on every change request.
	Possible attributes are id, target, branch, fork, url, title, author, authorDisplayName, and authorEmail
		
	Example:
 
```
	when { changeRequest() }
	when { changeRequest target: 'master' }.
```
	
	- EQUALS for a simple string comparison (the default)
	- GLOB for an ANT style path glob (same as for example changeset)
	- REGEXP for regular expression matching
	  Example:
   
```
   	when { changeRequest authorEmail: "[\\w_-.]+@example.com", comparator: 'REGEXP' }
```

6. environment - Execute the stage when the specified environment variable is set to the given value
	Example: when { environment name: 'DEPLOY_TO', value: 'production' }.
	
7. equals - Execute the stage when the expected value is equal to the actual value 
	Example: when { equals expected: 2, actual: currentBuild.number }
		
8. expression - Execute the stage when the specified Groovy expression evaluates to true
       Example:
       
```
	when { expression { return params.DEBUG_BUILD } }

	stage {
	      when {
	          expression { BRANCH_NAME ==~ /(production|staging)/ }
	          anyOf {
		  environment name: 'DEPLOY_TO', value: 'production'
	          environment name: 'DEPLOY_TO', value: 'staging'
		  }
	   }
	}
```

10. tag : Execute the stage if the TAG_NAME variable matches the given pattern. If an empty pattern is provided the stage will execute if the TAG_NAME variable exists (same as buildingTag()).
	Example:
 
```
	when { tag "release-*" }
```
	
11. not - Execute the stage when the nested condition is false. Must contain one condition. 
	Example:
 
```
 	when { not { branch 'master' } }
```
	
12. allOf - Execute the stage when all of the nested conditions are true
	Example:
 
```
	when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }
```

13. anyOf - Execute the stage when at least one of the nested conditions is true. Must contain at least one condition.
	Example:
 
```
	when { anyOf { branch 'master'; branch 'staging' } }
```
		
14. triggeredBy - Execute the stage when the current build has been triggered by the param given
	Example:
 
```
	when { triggeredBy 'SCMTrigger' }
	when { triggeredBy 'TimerTrigger' }
	when { triggeredBy 'BuildUpstreamCause' }
	when { triggeredBy cause: "UserIdCause", detail: "vlinde" }
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


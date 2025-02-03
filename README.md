## Jenkins
eclarative Pipeline:
Enclosed in :
  	pipeline {
  	}

The top-level of the Pipeline must be a block, specifically: pipeline { }.

No semicolons as statement separators. Each statement has to be on its own line.

Blocks must only consist of Sections, Directives, Steps, or assignment statements.

A property reference statement is treated as a no-argument method invocation. So, for example, input is treated as input().

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Sections in pipeline:**

1. agent - mandatory
   Two ways to declare agents -
     1. Top Level Agents - declared at the top level of a Pipeline, an agent is allocated and then the timeout option is applied
     2. Stage Agents - declared within a stage, the options are invoked before allocating the agent and before checking any when conditions.

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

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


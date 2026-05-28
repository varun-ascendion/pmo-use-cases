# Epic Conformance Table



|     Epic   ID    	|     Check   Area                 	|     What   “Good” Looks Like                                             	|     Status   (Yes / Partially / No)    	|     Notes   / Gaps    	|
|------------------	|----------------------------------	|--------------------------------------------------------------------------	|----------------------------------------	|-----------------------	|
|     EPIC-123     	|     Clear   Problem Statement    	|     States   current technical limitation and why it matters             	|                                        	|                       	|
|     EPIC-123     	|     Defined   Outcome / Goal     	|     Describes   measurable improvement (not tasks)                       	|                                        	|                       	|
|     EPIC-123     	|     In Scope   Defined           	|     Explicit   inclusions listed                                         	|                                        	|                       	|
|     EPIC-123     	|     Out of   Scope Defined       	|     Explicit   exclusions listed                                         	|                                        	|                       	|
|     EPIC-123     	|     Business /   Tech Value      	|     Explains   reliability, scale, cost, security, or velocity impact    	|                                        	|                       	|
|     EPIC-123     	|     Success   Metrics            	|     Objective,   testable completion criteria                            	|                                        	|                       	|
|     EPIC-123     	|     Dependencies   Identified    	|     External   teams/systems called out                                  	|                                        	|                       	|
|     EPIC-123     	|     Risks   Identified           	|     Migration,   perf, compatibility risks stated                        	|                                        	|                       	|
|     EPIC-123     	|     NFRs   Documented            	|     Perf,   security, availability, observability                        	|                                        	|                       	|
|     EPIC-123     	|     Rollout   Strategy           	|     Canary,   phased, rollback defined                                   	|                                        	|                       	|
|     EPIC-123     	|     Stories   Properly Linked    	|     All   delivery work tracked via child issues                         	|                                        	|                       	|


---

# User Story Conformance Table

|     Story   ID    	|     Epic   ID    	|     Check   Area                        	|     What   “Good” Looks Like                          	|     Status   (Yes / Partially / No)    	|     Notes   / Gaps    	|
|-------------------	|------------------	|-----------------------------------------	|-------------------------------------------------------	|----------------------------------------	|-----------------------	|
|     STORY-456     	|     EPIC-123     	|     Clear   System-Oriented Story       	|     “As a   system/service…” or equivalent            	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Epic   Alignment                    	|     Story   clearly contributes to Epic outcome       	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Context   Provided                  	|     Current vs   desired behavior explained           	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Functional   Acceptance Criteria    	|     Given/When/Then   or equivalent                   	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Non-Functional   Criteria           	|     Perf,   security, compatibility, observability    	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Implementation   Notes              	|     Constraints/guidance   (not task list)            	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Dependencies   Declared             	|     Explicit   blockers listed                        	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Test   Expectations                 	|     Unit/integration/perf   expectations defined      	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Definition   of Done                	|     Clear,   consistent with team DoD                 	|                                        	|                       	|
|     STORY-456     	|     EPIC-123     	|     Appropriately   Sized               	|     Small   enough to complete in one sprint          	|                                        	|                       	|


---

# Epic to User Story Alignment

|     Epic   ID    	|     Alignment   Check                            	|     Pass   (Yes / No)    	|     Notes    	|
|------------------	|--------------------------------------------------	|--------------------------	|--------------	|
|     EPIC-123     	|     All success   metrics covered by ≥1 story    	|                          	|              	|
|     EPIC-123     	|     NFRs   traceable to stories                  	|                          	|              	|
|     EPIC-123     	|     No stories   unrelated to Epic goal          	|                          	|              	|
|     EPIC-123     	|     Migration /   rollout work represented       	|                          	|              	|
|     EPIC-123     	|     Cleanup /   deprecation work included        	|                          	|              	|
<!--
 Licensed to the Apache Software Foundation (ASF) under one
 or more contributor license agreements.  See the NOTICE file
 distributed with this work for additional information
 regarding copyright ownership.  The ASF licenses this file
 to you under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance
 with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing,
 software distributed under the License is distributed on an
 "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 KIND, either express or implied.  See the License for the
 specific language governing permissions and limitations
 under the License.
 -->

# CI Sequence diagrams

You can see here the sequence diagrams of the flow happening during the CI Jobs.

## Pull request flow from fork

```mermaid
sequenceDiagram
    Note over Airflow Repo: pull request
    Note over Tests: pull_request<br>[Read Token]
    Note over Build Images: pull_request_target<br>[Write Token]
    activate Airflow Repo
    Airflow Repo -->> Tests: Trigger 'pull_request'
    activate Tests
    Tests -->> Build Images: Trigger 'pull_request_target'
    activate Build Images
    Note over Build Images: Build info<br>Decide which Python
    Note over Tests: Build info<br>Decide on tests<br>Decide on Matrix (selective)
    Note over Tests: Skip Build<br>(Runs in 'Build Images')<br>CI Images
    Note over Tests: Skip Build<br>(Runs in 'Build Images')<br>PROD Images
    par
        Note over Build Images: Build CI Images<br>[COMMIT_SHA]<br>Use latest constraints<br>or upgrade if setup changed
    and
        Note over Tests: OpenAPI client gen
    and
        Note over Tests: React WWW tests
    and
        Note over Tests: Test examples<br>PROD image building
    and
        Note over Tests: Test git clone on Windows
    and
        opt
            Note over Tests: Run basic <br>static checks
        end
    end
    Build Images ->> GitHub Registry: Push CI Images<br>[COMMIT_SHA]
    loop Wait for CI images
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    end
    Note over Tests: Verify CI Images<br>[COMMIT_SHA]
    par
        GitHub Registry ->> Build Images: Pull PROD Images<br>[latest]
        Note over Build Images: Build PROD Images<br>[COMMIT_SHA]
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Run static checks
        end
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Build docs
        end
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Test Pytest collection<br>[COMMIT_SHA]
            par
                opt
                    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                    Note over Tests: Unit Tests<br>Python/DB matrix
                end
            and
                opt
                     GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                     Note over Tests: Integration Tests
                end
            and
                opt
                     GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                     Note over Tests: Quarantined Tests
                end
            end
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Test provider <br>packages build
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Test airflow <br>packages build
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Helm tests
        end
    end
    Note over Tests: Summarize Warnings
    Build Images ->> GitHub Registry: Push PROD Images<br>[COMMIT_SHA]
    deactivate Build Images
    loop Wait for PROD images
        GitHub Registry ->> Tests: Pull PROD Images<br>[COMMIT_SHA]
    end
    Note over Tests: Verify PROD Image<br>[COMMIT_SHA]
    par
        opt
            GitHub Registry ->> Tests: Pull PROD Images<br>[COMMIT_SHA]
            Note over Tests: Run Kubernetes <br>tests
        end
    and
        opt
            GitHub Registry ->> Tests: Pull PROD Images<br>[COMMIT_SHA]
            Note over Tests: Run docker-compose <br>tests
        end
    end
    opt
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Generate constraints
    end
    Note over Tests: Build ARM CI images
    Tests -->> Airflow Repo: Status update
    deactivate Airflow Repo
    deactivate Tests
```

## Pull request flow from "apache/airflow" repo

```mermaid
sequenceDiagram
    Note over Airflow Repo: pull request
    Note over Tests: pull_request<br>[Read Token]
    Note over Build Images: pull_request_target<br>[Write Token]
    activate Airflow Repo
    Airflow Repo -->> Tests: Trigger 'pull_request'
    activate Tests
    Tests -->> Build Images: Trigger 'pull_request_target'
    activate Build Images
    Note over Build Images: Build info
    Note over Build Images: Skip Build<br>(Runs in 'Tests')<br>CI Images
    Note over Build Images: Skip Build<br>(Runs in 'Tests')<br>PROD Images
    deactivate Build Images
    Note over Tests: Build info<br>Decide on tests<br>Decide on Matrix (selective)
    par
        Note over Tests: Build CI Images<br>[COMMIT_SHA]<br>Use latest constraints<br>or upgrade if setup changed
    and
        Note over Tests: OpenAPI client gen
    and
        Note over Tests: Test UI
    and
        Note over Tests: Test examples<br>PROD image building
    and
        Note over Tests: Test git clone on Windows
    and
        opt
            Note over Tests: Run basic <br>static checks
        end
    end
    Tests ->> GitHub Registry: Push CI Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    Note over Tests: Verify CI Image<br>[COMMIT_SHA]
    par
        Note over Tests: Build PROD Images<br>[COMMIT_SHA]
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Run static checks
        end
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Build docs
        end
    and
        opt
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Test Pytest collection<br>[COMMIT_SHA]
            par
                opt
                    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                    Note over Tests: Unit Tests<br>Python/DB matrix/No DB
                end
            and
                opt
                     GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                     Note over Tests: Integration Tests
                end
            and
                opt
                     GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
                     Note over Tests: Quarantined Tests
                end
            end
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Test provider <br>packages build
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Test airflow <br>packages build
        end
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Helm tests
        end
    end
    Note over Tests: Summarize Warnings
    Tests ->> GitHub Registry: Push PROD Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull PROD Image<br>[COMIT_SHA]
    Note over Tests: Verify PROD Image<br>[COMMIT_SHA]
    par
        opt
            GitHub Registry ->> Tests: Pull PROD Images<br>[COMMIT_SHA]
            Note over Tests: Run Kubernetes <br>tests
        end
    and
        opt
            GitHub Registry ->> Tests: Pull PROD Images<br>[COMMIT_SHA]
            Note over Tests: Run docker-compose <br>tests
        end
    end
    opt
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Generate constraints
    end
    Note over Tests: Build ARM CI images
    Tests -->> Airflow Repo: Status update
    deactivate Airflow Repo
    deactivate Tests
```

## Merge "Canary" run

```mermaid
sequenceDiagram
    Note over Airflow Repo: push/merge
    Note over Tests: push<br>[Write Token]
    activate Airflow Repo
    Airflow Repo -->> Tests: Trigger 'push'
    activate Tests
    Note over Tests: Build info<br>All tests<br>Full matrix
    par
        Note over Tests: Build CI Images<br>[COMMIT_SHA]<br>Always upgrade deps
    and
        Note over Tests: Check that image builds quickly
    and
        Note over Tests: Push early cache and images
    and
        Note over Tests: OpenAPI client gen
    and
        Note over Tests: React WWW tests
    and
        Note over Tests: Test examples<br>PROD image building
    and
        Note over Tests: Test git clone on Windows
    end
    Tests ->> GitHub Registry: Push CI Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    Note over Tests: Verify CI Image<br>[COMMIT_SHA]
    par
        Note over Tests: Build PROD Images<br>[COMMIT_SHA]
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Run static checks
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Build docs
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Test Pytest collection<br>[COMMIT_SHA]
        par
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Unit Tests<br>Python/DB matrix
        and
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Integration Tests
        and
           GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
           Note over Tests: Quarantined Tests
        end
    and
        opt
           GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
           Note over Tests: Test provider <br>packages build
        end
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Test airflow <br>packages build
    and
        opt
             GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
             Note over Tests: Helm tests
        end
    end
    Note over Tests: Summarize Warnings
    Tests ->> GitHub Registry: Push PROD Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
    Note over Tests: Verify PROD Image<br>[COMMIT_SHA]
    par
        GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
        Note over Tests: Run Kubernetes <br>tests
    and
        GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
        Note over Tests: Run docker-compose <br>tests
    end
    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    Note over Tests: Generate constraints
    Tests ->> Airflow Repo: Push constraints if changed
    Note over Tests: Build CI Images<br>[latest]<br>Use latest constraints
    Tests ->> GitHub Registry: Push CI Image<br>[latest]
    Note over Tests: Build PROD Images<br>[latest]<br>Use latest constraints
    Tests ->> GitHub Registry: Push PROD Image<br>[latest]
    Note over Tests: Build ARM CI images
    Tests ->> GitHub Registry: Push ARM CI Image cache
    Tests -->> Airflow Repo: Status update
    deactivate Airflow Repo
    deactivate Tests
```

## Scheduled run

```mermaid
sequenceDiagram
    Note over Airflow Repo: scheduled
    Note over Tests: push<br>[Write Token]
    activate Airflow Repo
    Airflow Repo -->> Tests: Trigger 'schedule'
    activate Tests
    Note over Tests: Build info<br>All tests<br>Full matrix
    par
        GitHub Registry ->> Tests: Pull CI Images<br>[latest]
        Note over Tests: Build CI Images<br>[COMMIT_SHA]<br>Always upgrade deps
    and
        Note over Tests: OpenAPI client gen
    and
        Note over Tests: React WWW tests
    and
        Note over Tests: Test examples<br>PROD image building
    and
        Note over Tests: Build CI Images<br>Use original constraints
    end
    Tests ->> GitHub Registry: Push CI Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    Note over Tests: Verify CI Image<br>[COMMIT_SHA]
    par
        GitHub Registry ->> Tests: Pull PROD Images<br>[latest]
        Note over Tests: Build PROD Images<br>[COMMIT_SHA]
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Run static checks
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Build docs
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Test Pytest collection<br>[COMMIT_SHA]
        par
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Unit Tests<br>Python/DB matrix
        and
            GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
            Note over Tests: Integration Tests
        and
           GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
           Note over Tests: Quarantined Tests
        end
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Test provider <br>packages build
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Test airflow <br>packages build
    and
        GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
        Note over Tests: Helm tests
    end
    Note over Tests: Summarize Warnings
    Tests ->> GitHub Registry: Push PROD Images<br>[COMMIT_SHA]
    GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
    Note over Tests: Verify PROD Image<br>[COMMIT_SHA]
    par
        GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
        Note over Tests: Run Kubernetes <br>tests
    and
        GitHub Registry ->> Tests: Pull PROD Image<br>[COMMIT_SHA]
        Note over Tests: Run docker-compose <br>tests
    end
    GitHub Registry ->> Tests: Pull CI Images<br>[COMMIT_SHA]
    Note over Tests: Generate constraints
    Tests ->> Airflow Repo: Push constraints if changed
    Note over Tests: Build CI Images<br>[latest]<br>Use latest constraints
    Tests ->> GitHub Registry: Push CI Image cache + latest
    Note over Tests: Build PROD Images<br>[latest]<br>Use latest constraints
    Tests ->> GitHub Registry: Push PROD Image cache + latest
    Note over Tests: Build ARM CI images
    Tests ->> GitHub Registry: Push ARM CI Image cache
    Tests -->> Airflow Repo: Status update
    deactivate Airflow Repo
    deactivate Tests
```

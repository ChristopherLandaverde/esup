#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Python version installed; we need 3.10-3.11
PYTHON=`command -v python3.11 || command -v python3.10`
DOCKER_COMPOSE = docker-compose
NON_DEV_FILE = docker-compose-non-dev.yml
CONTAINER_NAME = superset_app
SERVICE = my-service  # Replace with your service name if needed

# Default target
.PHONY: help
help:
	@echo "Available commands:"
	@echo "  make up           - Start dev containers in detached mode"
	@echo "  make up-non-dev   - Start non-dev containers in detached mode"
	@echo "  make down         - Stop dev containers"
	@echo "  make down-non-dev - Stop non-dev containers"
	@echo "  make build        - Build dev containers"
	@echo "  make build-non-dev - Build non-dev containers"
	@echo "  make logs         - View logs for dev containers"
	@echo "  make logs-non-dev - View logs for non-dev containers"
	@echo "  make restart      - Restart dev containers"
	@echo "  make restart-non-dev - Restart non-dev containers"
	@echo "  make exec         - Execute shell in a dev container"
	@echo "  make exec-non-dev - Execute shell in a non-dev container"
	@echo "  make ps           - List running dev containers"
	@echo "  make ps-non-dev   - List running non-dev containers"

# Dev environment
.PHONY: up
up:
	$(DOCKER_COMPOSE) up -d

.PHONY: down
down:
	$(DOCKER_COMPOSE) down

.PHONY: build
build:
	$(DOCKER_COMPOSE) build

.PHONY: logs
logs:
	$(DOCKER_COMPOSE) logs -f

.PHONY: restart
restart:
	$(DOCKER_COMPOSE) down && $(DOCKER_COMPOSE) up -d

.PHONY: exec
exec:
	$(DOCKER_COMPOSE) exec $(SERVICE) sh

.PHONY: ps
ps:
	$(DOCKER_COMPOSE) ps

# Non-dev environment
.PHONY: up-non-dev
up-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) up -d

.PHONY: down-non-dev
down-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) down

.PHONY: build-non-dev
build-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) build

.PHONY: logs-non-dev
logs-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) logs -f

.PHONY: restart-non-dev
restart-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) down && $(DOCKER_COMPOSE) -f $(NON_DEV_FILE) up -d

.PHONY: exec-non-dev
exec-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) exec $(SERVICE) sh

.PHONY: ps-non-dev
ps-non-dev:
	$(DOCKER_COMPOSE) -f $(NON_DEV_FILE) ps

# Docker exec to access Superset container
.PHONY: exec-superset
exec-superset:
	docker exec -it $(CONTAINER_NAME) bash






.PHONY: install superset venv pre-commit

install: superset pre-commit

superset:
	# Install external dependencies
	pip install -r requirements/development.txt

	# Install Superset in editable (development) mode
	pip install -e .

	# Create an admin user in your metadata database
	superset fab create-admin \
                    --username admin \
                    --firstname "Admin I."\
                    --lastname Strator \
                    --email admin@superset.io \
                    --password general

	# Initialize the database
	superset db upgrade

	# Create default roles and permissions
	superset init

	# Load some data to play with
	superset load-examples

	# Install node packages
	cd superset-frontend; npm ci

update: update-py update-js

update-py:
	# Install external dependencies
	pip install -r requirements/development.txt

	# Install Superset in editable (development) mode
	pip install -e .

	# Initialize the database
	superset db upgrade

	# Create default roles and permissions
	superset init

update-js:
	# Install js packages
	cd superset-frontend; npm ci

venv:
	# Create a virtual environment and activate it (recommended)
	if ! [ -x "${PYTHON}" ]; then echo "You need Python 3.10 or 3.11 installed"; exit 1; fi
	test -d venv || ${PYTHON} -m venv venv # setup a python3 virtualenv
	. venv/bin/activate

activate:
	. venv/bin/activate

pre-commit:
	# setup pre commit dependencies
	pip3 install -r requirements/development.txt
	pre-commit install

format: py-format js-format

py-format: pre-commit
	pre-commit run black --all-files

py-lint: pre-commit
	pylint -j 0 superset

js-format:
	cd superset-frontend; npm run prettier

flask-app:
	flask run -p 8088 --with-threads --reload --debugger

node-app:
	cd superset-frontend; npm run dev-server

build-cypress:
	cd superset-frontend; npm run build-instrumented
	cd superset-frontend/cypress-base; npm ci

open-cypress:
	if ! [ $(port) ]; then cd superset-frontend/cypress-base; CYPRESS_BASE_URL=http://localhost:9000 npm run cypress open; fi
	cd superset-frontend/cypress-base; CYPRESS_BASE_URL=http://localhost:$(port) npm run cypress open

report-celery-worker:
	celery --app=superset.tasks.celery_app:app worker

report-celery-beat:
	celery --app=superset.tasks.celery_app:app beat --pidfile /tmp/celerybeat.pid --schedule /tmp/celerybeat-schedulecd

admin-user:
	superset fab create-admin

# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM python:3.12.6-slim@sha256:ad48727987b259854d52241fac3bc633574364867b8e20aec305e6e7f4028b26 AS base

FROM base AS builder

COPY requirements.txt .

RUN pip install --prefix="/install" -r requirements.txt

FROM base

WORKDIR /loadgen

COPY --from=builder /install /usr/local

# Add application code
COPY locustfile.py .

# Create directory for results
RUN mkdir -p /loadgen/results

# enable gevent support in debugger
ENV GEVENT_SUPPORT=True

# Default to worker mode, can be overridden
ENV LOCUST_MODE=worker
ENV LOCUST_MASTER_HOST=localhost
ENV LOCUST_MASTER_PORT=5557

ENTRYPOINT if [ "$LOCUST_MODE" = "master" ]; then \
    locust --master --host="http://${FRONTEND_ADDR}" --web-port=8089; \
  elif [ "$LOCUST_MODE" = "worker" ]; then \
    locust --worker --master-host=${LOCUST_MASTER_HOST} --master-port=${LOCUST_MASTER_PORT} --host="http://${FRONTEND_ADDR}"; \
  else \
    locust --host="http://${FRONTEND_ADDR}" --headless -u "${USERS:-10}"; \
  fi

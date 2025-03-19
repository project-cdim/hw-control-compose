# syntax=docker/dockerfile:1
ARG PYTHON_BASE=3.12
# build stage
FROM python:$PYTHON_BASE AS builder

# install PDM
RUN pip install --no-cache-dir -U pdm==2.22.2
# disable update check
ENV PDM_CHECK_UPDATE=false
# copy files
COPY ./hw-control/hw-control/pyproject.toml /project/
COPY ./hw-control/hw-control/pdm.lock /project/
COPY ./hw-control/hw-control/README.md /project/
COPY ./hw-control/hw-control/LICENSE /project/

# install dependencies and project into the local packages directory
WORKDIR /project
RUN git config --global http.sslverify false && git config --global credential.helper store
RUN pdm install --check --prod

# build stage
FROM python:$PYTHON_BASE AS src

COPY ./hw-control/hw-control/src/ /app/src/
COPY ./hw-control/oob-plugin-reference/src/ /app/src/
COPY ./hw-control/fm-plugin-reference/src/ /app/src/

# run stage
FROM python:$PYTHON_BASE

# make default directories
RUN mkdir -p /var/log/cdim
RUN mkdir -p /opt/cdim/hw-control/data

# make default config directory with default settings
COPY ./hw-control/hw-control/config /opt/cdim/hw-control/config

# retrieve packages from build stage
COPY --from=builder /project/.venv/ /project/.venv
COPY --from=src /app/src/ /project/src/
COPY ./configs/oob-plugin/ /project/src/plugins/oob/
COPY ./configs/fm-plugin/ /project/src/plugins/fm/

ENV PATH="/project/.venv/bin:$PATH"
# if use .py instead of .pyc, uncomment next 2 lines.
WORKDIR /project/src
# set command/entrypoint, adapt to fit your needs
CMD ["uvicorn", "app.hw_control_main:app", "--host", "0.0.0.0", "--port", "8000"]

FROM illumidesk/r-workspace:latest

COPY requirements.txt /tmp/
RUN pip install -r /tmp/requirements.txt

USER root

RUN curl -fsSL https://code-server.dev/install.sh | sh

RUN fix-permissions "${HOME}" \
 && fix-permissions "${CONDA_DIR}" \
 && fix-permissions /etc/jupyter

USER "${NB_USER}"

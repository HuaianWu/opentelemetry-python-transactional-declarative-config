FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=53a5a40c9604583c501bcf13970a635f00e62df4
RUN git clone https://github.com/open-telemetry/opentelemetry-python . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# uv workspace: the repo root is not an installable package, so the workspace members are installed
# editable from uv.lock (`--all-packages`), plus the repo's own pinned dev-requirements.txt (pytest etc.),
# which is what tox-uv uses in upstream CI. mars-base's uv 0.9.18 cannot read this lockfile.
ENV UV_PROJECT_ENVIRONMENT=/opt/venv
RUN pip install --no-cache-dir "uv==0.12.9" \
    && python -m uv sync --frozen --all-packages --no-default-groups \
    && python -m uv pip install --python /opt/venv/bin/python -r dev-requirements.txt \
    && test -z "$(git status --porcelain)"
ENV PATH="/opt/venv/bin:${PATH}"

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]

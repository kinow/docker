# NOTE: until DVC offers an image; https://github.com/iterative/dvc/discussions/5934
#       Tested with apptainer, then singularity on BSC MareNostrum5 and on CSC LUMI

FROM python:3.12-slim-bookworm

RUN pip install -U pip && \
    pip install dvc

CMD ["dvc"]

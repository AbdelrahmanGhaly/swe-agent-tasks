FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir -e ".[dev]" \
    && chmod -R a+rwX /app

ENV MPLBACKEND=Agg
ENV PYTEST_ADDOPTS="--deselect tests/test_annotation.py::TestAnnotation::test_1 --deselect tests/test_annotation.py::TestAnnotation::test_2 --deselect tests/test_annotation.py::TestAnnotation::test_3 --deselect tests/test_record.py::TestRecord::test_1a --deselect tests/test_record.py::TestRecord::test_1b --deselect tests/test_record.py::TestRecord::test_1c --deselect tests/test_record.py::TestRecord::test_1d --deselect tests/test_record.py::TestRecord::test_1e --deselect tests/test_record.py::TestRecord::test_2a --deselect tests/test_record.py::TestRecord::test_2b --deselect tests/test_record.py::TestRecord::test_3a --deselect tests/test_record.py::TestRecord::test_3b --deselect tests/test_record.py::TestRecord::test_4b --deselect tests/test_record.py::TestRecord::test_4c --deselect tests/test_record.py::TestMultiRecord::test_multi_variable_d --deselect tests/test_record.py::TestDownload::test_dl_database_no_dat_file --deselect tests/test_record.py::TestDownload::test_dl_database_with_dat_file"

CMD ["/bin/bash"]

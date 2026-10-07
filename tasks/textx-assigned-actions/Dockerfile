FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV UV_PROJECT_ENVIRONMENT=/opt/venv \
    UV_LINK_MODE=copy \
    VIRTUAL_ENV=/opt/venv \
    PATH="/opt/venv/bin:${PATH}"

RUN uv sync --locked \
    && chmod -R a+rX /opt/venv

# The functional suite writes .dot exports and generated sources into the source
# tree. Pre-create them and keep the source tree writable so the suite can run
# under any uid.
RUN mkdir -p examples/Entity/srcgen examples/Entity/dotexport \
    && touch \
        Model_parse_tree.dot Model_parser_model.dot \
        import_test_mm.dot import_test_model.dot \
        multipleimport_test_mm.dot multipleimport_test_model.dot \
        test_issue_14_metamodel.dot test_issue_14_model.dot \
        textx_model_parse_tree.dot textx_model_parser_model.dot \
        examples/Entity/dotexport/entity_meta.dot examples/Entity/dotexport/person.dot \
        examples/Entity/srcgen/Address.java examples/Entity/srcgen/Person.java \
        examples/hello_world/example.dot examples/hello_world/hello_meta.dot \
        examples/pyFlies/experiment.dot examples/pyFlies/pyflies_meta.dot \
        examples/robot/program.dot examples/robot/robot_meta.dot \
        examples/workflow/example.dot examples/workflow/workflow_meta.dot \
        tests/functional/registration/projects/flow_dsl/tests/models/data_flow.pu \
        tests/functional/test_metamodel/model_param_generate_test.testtarget \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]

FROM public.ecr.aws/d3j8x8q7/olympus-base-python:latest

WORKDIR /app
COPY . .

ENV PYTHONDONTWRITEBYTECODE=1

RUN pip install --no-cache-dir \
        numpy==1.26.4 \
        pandas==2.2.3 \
        jplephem==2.24 \
        sgp4==2.27 \
        certifi==2026.5.20 \
        pytz==2026.4 \
        pyflakes==3.2.0 \
        pytest==9.0.3 \
        "assay @ https://github.com/brandon-rhodes/assay/archive/74617d70e77afa09f58b3169cf496679ac5d5621.zip" \
    && pip install --no-cache-dir --no-deps -e . \
    && git config --system --add safe.directory '*' \
    && printf '%s\n' \
        'import inspect' \
        'import os' \
        '' \
        'import pytest' \
        'from assay.runner import find_fixture, generate_arguments_from_fixtures' \
        '' \
        'collect_ignore = ["contrib", "design"]' \
        '' \
        '' \
        'def pytest_configure(config):' \
        '    os.chdir(os.path.join(os.path.dirname(os.path.abspath(__file__)), "ci"))' \
        '' \
        '' \
        'def pytest_pycollect_makeitem(collector, name, obj):' \
        '    if not (inspect.isfunction(obj) and collector.funcnamefilter(name)):' \
        '        return None' \
        '    names = inspect.getfullargspec(obj).args' \
        '    if not names:' \
        '        return None' \
        '    module = collector.obj' \
        '' \
        '    def run():' \
        '        fixtures = [find_fixture(module, n) for n in names]' \
        '        for args in generate_arguments_from_fixtures(names, fixtures):' \
        '            obj(*args)' \
        '' \
        '    run.__name__ = name' \
        '    return pytest.Function.from_parent(collector, name=name, callobj=run)' \
        > conftest.py \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]

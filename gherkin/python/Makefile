SHELL := /usr/bin/env bash
GOOD_FEATURE_FILES = $(shell find testdata/good -name "*.feature")
BAD_FEATURE_FILES  = $(shell find testdata/bad -name "*.feature")

TOKENS   = $(patsubst testdata/%.feature,acceptance/testdata/%.feature.tokens,$(GOOD_FEATURE_FILES))
ASTS     = $(patsubst testdata/%.feature,acceptance/testdata/%.feature.ast.ndjson,$(GOOD_FEATURE_FILES))
PICKLES  = $(patsubst testdata/%.feature,acceptance/testdata/%.feature.pickles.ndjson,$(GOOD_FEATURE_FILES))
SOURCES  = $(patsubst testdata/%.feature,acceptance/testdata/%.feature.source.ndjson,$(GOOD_FEATURE_FILES))
ERRORS   = $(patsubst testdata/%.feature,acceptance/testdata/%.feature.errors.ndjson,$(BAD_FEATURE_FILES))

PYTHON_FILES = $(shell find . -name "*.py")

.DELETE_ON_ERROR:

default: .compared
.PHONY: all

.compared: .built $(TOKENS) $(ASTS) $(PICKLES) $(ERRORS) $(SOURCES)
	touch $@

.built: gherkin/parser.py gherkin/gherkin-languages.json $(PYTHON_FILES) bin/gherkin .pipped
	nosetests
	touch $@

.pipped:
	pip install -r requirements.txt
	touch $@

acceptance/testdata/%.feature.tokens: testdata/%.feature testdata/%.feature.tokens .built
	mkdir -p `dirname $@`
	bin/gherkin-generate-tokens $< > $@
	diff --unified $<.tokens $@

acceptance/testdata/%.feature.ast.ndjson: testdata/%.feature testdata/%.feature.ast.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-source --no-pickles $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.ast.ndjson) <(jq "." $@)

acceptance/testdata/%.feature.pickles.ndjson: testdata/%.feature testdata/%.feature.pickles.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-source --no-ast $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.pickles.ndjson) <(jq "." $@)

acceptance/testdata/%.feature.source.ndjson: testdata/%.feature testdata/%.feature.source.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-ast --no-pickles $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.source.ndjson) <(jq "." $@)

acceptance/testdata/%.feature.errors.ndjson: testdata/%.feature testdata/%.feature.errors.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-source $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.errors.ndjson) <(jq "." $@)

clean:
	rm -rf .compared .built .pipped acceptance
.PHONY: clean

clobber: clean
	rm -rf gherkin/parser.py
.PHONY: clobber

gherkin/parser.py: gherkin.berp gherkin-python.razor berp/berp.exe
	-mono berp/berp.exe -g gherkin.berp -t gherkin-python.razor -o $@
	# Remove BOM
	awk 'NR==1{sub(/^\xef\xbb\xbf/,"")}{print}' < $@ > $@.nobom
	mv $@.nobom $@

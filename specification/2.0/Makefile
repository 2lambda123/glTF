# Copyright 2020 The Khronos Group Inc.
# SPDX-License-Identifier: Apache-2.0

# Output specification targets
SPEC = Specification
TARGETS = $(SPEC).html $(SPEC).pdf
all: $(TARGETS)

# Generate the Properties Reference section of the spec from JSON schema.
# This uses 'npx' to minimally install the 'wetzel' tool, if needed.
# When using embedded schemas, SCHEMALINK should be a relative path.

# Note this Makefile should remain locked to a particular version of
# wetzel, such that the history of changes to the documentation can
# accurately reflect changes to wetzel that impact the output here.
# Please manually upgrade the version as often as needed, without unlocking.

WETZEL = npx wetzel@0.2.2
SCHEMALINK = schema
EMBEDSCHEMA = JsonSchemaReference.adoc

# Base name of the generated properties reference file
PROPREF = PropertiesReference.adoc
# Files generated by wetzel
GENERATED = $(PROPREF) $(EMBEDSCHEMA)
# Schema files to leave out of $(PROPREF)
IGNORESCHEMA = '["gltfchildofrootproperty.schema.json", "gltfid.schema.json", "gltfproperty.schema.json"]'
$(GENERATED): $(wildcard schema/*.json)
	$(WETZEL) -n -a=cqo -m=a -p "$(SCHEMALINK)" -e "$(EMBEDSCHEMA)" \
	    -i $(IGNORESCHEMA) -c "icon:check[]" -k "**MUST**"\
	    schema/glTF.schema.json > $(PROPREF)

# Spec targets for offline generation
# Requires an up-to-date asciidoctor, asciidoctor-pdf, and
# asciidoctor-mathematical be installed
# We recommend using the following image on dockerhub:
# docker pull khronosgroup/docker-images:asciidoctor-spec
ASCIIDOCTOR = asciidoctor $(ADOCOPTS)
ADOCOPTS = -d book
ADOCHTMLOPTS = -a stylesheet=khronos.css -a sectanchors
SPECDEPS = $(SPEC).adoc $(GENERATED)

PATCHVERSION = 1
SPECREVISION = 2.0.$(PATCHVERSION)

# Spell out ISO 8601 format as not all date commands support --rfc-3339
SPECDATE = $(shell echo `date -u "+%Y-%m-%d %TZ"`)

# Generate AsciiDoc attributes for spec remark
# Could use `git log -1 --format="%cd"` to get branch commit date
# This used to be a dependency in the spec html/pdf targets,
# but that's likely to lead to merge conflicts. Just regenerate
# when pushing a new spec for review to the sandbox.
# The dependency on HEAD is per the suggestion in
# http://neugierig.org/software/blog/2014/11/binary-revisions.html
SPECREMARK = from git branch: $(shell echo `git symbolic-ref --short HEAD 2> /dev/null || echo Git branch not available`) \
             commit: $(shell echo `git log -1 --format="%H" 2> /dev/null || echo Git commit not available`)

ATTRIBOPTS = -a revnumber="$(SPECREVISION)" \
             -a revdate="$(SPECDATE)" \
             -a revremark="$(SPECREMARK)"

$(SPEC).html: $(SPECDEPS)
	$(ASCIIDOCTOR) -b html5 $(ADOCHTMLOPTS) $(ATTRIBOPTS) $(SPEC).adoc -o $@

# :allow-url-read: is necessary for the embedded render.githubusers.com
# math images to be processed for the PDF target. See
# https://github.com/asciidoctor/asciidoctor-pdf/issues/369
# asciidoctor-mathematical leaves intermediate images of equations behing.
# These are removed after creating the PDF.
STEMIMAGES = stem-*.png
$(SPEC).pdf: $(SPECDEPS)
	$(ASCIIDOCTOR) -b pdf $(ATTRIBOPTS) -a allow-uri-read -r asciidoctor-pdf \
	    -r asciidoctor-mathematical $(SPEC).adoc -o $@
	rm -f $(STEMIMAGES)

clean:
	-rm -f $(GENERATED) $(TARGETS)

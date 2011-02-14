include ../Makefile.inc

CSS = RunDeck-Guide.css
DIST = dist

.PHONY: html clean

all : dist html figures

dist :
	mkdir -p $(DIST)

$(DIST)/html : dist
	mkdir -p $(DIST)/html

html : $(DIST)/html
	$(ECHO) Generating HTML documents
	$(PANDOC) --number-sections --toc -s title.txt 0[0-9]-*/01-*.md --css=$(CSS) --template=templates/html.template --include-before=templates/before.html --include-after=templates/after.html -o $(DIST)/html/RunDeck-Training.html
	$(PANDOC) --number-sections --toc -s title.txt 0[0-9]-*/tutorial.md --css=$(CSS) --template=templates/html.template --include-before=templates/before.html --include-after=templates/after.html -o $(DIST)/html/RunDeck-Tutorial.html
	cp -f $(CSS) $(DIST)/html

$(DIST)/html/figures : dist
	mkdir -p $(DIST)/html/figures

figures : $(DIST)/html/figures
	- cp -f ../figures/*.png $(DIST)/html/figures

clean :
	rm -fr $(DIST)

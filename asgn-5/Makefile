PROJECT = report
PDF = $(PROJECT).pdf
DOC = document.tex

watch: latexmk

latexmk: $(DOC)
	latexmk $(DOC) \
		-pvc \
		-pdf \
		-e '$$pdflatex=q/xelatex %O -interaction=nonstopmode %S/'

report: $(PDF)

$(PDF): $(NOTES)
	pandoc $(NOTES) \
		-o $(PDF) \
		--latex-engine=xelatex \
		--variable urlcolor=cyan \
		-V papersize:a4 \
		-V geometry:margin=1.5in \
		--filter pandoc-citeproc \
		--template=template.tex

clean:
	rm *.aux \
		*.fdb_latexmk \
		*.fls \
		*.log


dist: report
	tar \
		--transform "s/^/$(PROJECT)\//" \
		-zcvf $(PROJECT).tar.gz \
		$(PDF)

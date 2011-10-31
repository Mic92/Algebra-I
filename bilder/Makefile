# Wandelt svg in pdf um
# es wird inkscape benötigt!
%.pdf: %.svg; inkscape --file='$<' --export-area-drawing --without-gui --export-pdf='$@'
SVG = $(wildcard *.svg)
PDF = $(patsubst %.svg,%.pdf,$(SVG))

all:  $(PDF) 
.PHONEY: clean
clean:
	rm -f *.pdf

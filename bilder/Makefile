# Wandelt svg in png um
# es wird convert von imagemagick benötigt!
%.png: %.svg; convert -background none '$<' '$@'
SVG = $(wildcard *.svg)
PNG = $(patsubst %.svg,%.png,$(SVG))

all: $(PNG)
clean:
	rm -f $(PNG)

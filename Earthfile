VERSION --arg-scope-and-set 0.7

kikit:
  FROM fedora:latest

  RUN yum -y install kicad python3-pip python3-mistune inkscape git zip librsvg2
  RUN python3 -m pip install --upgrade pip && pip install kikit && pip install git+https://github.com/yaqwsx/PcbDraw.git@4699cfd90aaa20d2fdff0f4c7ee6ac9d7a9d8758

  ARG BRANCH=main
  SAVE IMAGE --cache-from=ghcr.io/daravis/kikit:main --push ghcr.io/daravis/kikit:$BRANCH

pollysdr-base:
  FROM ghcr.io/daravis/kikit:main

  WORKDIR /workdir
  COPY . .
  RUN kikit drc run projects/pollysdr/pollysdr_pcb/pollysdr_pcb.kicad_pcb

pollysdr-panelize:
  FROM +pollysdr-base

  IF $(git status --porcelain=v1 2>/dev/null)
    LET REVISION="$(git rev-parse --abbrev-ref HEAD)-$(git rev-parse --short HEAD)~"
  ELSE
    LET REVISION="$(git rev-parse --abbrev-ref HEAD)-$(git rev-parse --short HEAD)"
  END
  
  RUN bash -c 'kikit panelize \
    --layout "grid; rows: 4; cols: 3; space: 3mm; hbackbone: 3mm; hbonecut: true" \
    --tabs "fixed; width: 3mm; vcount: 2; hcount: 0" \
    --cuts vcuts \
    --framing "tightframe; width: 5mm; space: 3mm; mintotalheight: 100mm; mintotalwidth: 100mm" \
    --post "millradius: 1mm" \
    --tooling "3hole; hoffset: 2.5mm; voffset: 2.5mm; size: 1.5mm" \
    --fiducials "3fid; hoffset: 5mm; voffset: 2.5mm; coppersize: 2mm; opening: 1mm;" \
    --text "simple; text: PollySDR; anchor: mt; voffset: 2.5mm; hjustify: center; vjustify: center;" \
    --text2 "simple; text: ${REVISION}; anchor: mb; voffset: -2.5mm; hjustify: center; vjustify: center;" \
    projects/pollysdr/pollysdr_pcb/pollysdr_pcb.kicad_pcb panel.kicad_pcb'
  SAVE ARTIFACT panel.kicad_pcb

pollysdr-panel-gerbers:
  FROM +pollysdr-base

  COPY +pollysdr-panelize/panel.kicad_pcb build/pollysdr_panel.kicad_pcb

  RUN mkdir -p build/outputs && kikit fab jlcpcb --no-drc build/pollysdr_panel.kicad_pcb build/outputs

  IF --no-cache [[ $(git status --porcelain=v1 2>/dev/null) ]]
    SAVE ARTIFACT build/outputs/gerbers.zip gerbers.zip AS LOCAL build/pollysdr-panel-dirty-$(git rev-parse --short HEAD).zip
  ELSE
    SAVE ARTIFACT build/outputs/gerbers.zip gerbers.zip AS LOCAL build/pollysdr-panel-$(git rev-parse --short HEAD).zip
  END


pollysdr-gerbers:
  FROM +pollysdr-base

  RUN mkdir -p build/outputs && kikit fab jlcpcb --no-drc projects/pollysdr/pollysdr_pcb/pollysdr_pcb.kicad_pcb build/outputs

  IF --no-cache [[ $(git status --porcelain=v1 2>/dev/null) ]]
    SAVE ARTIFACT build/outputs/gerbers.zip gerbers.zip AS LOCAL build/pollysdr-dirty-$(git rev-parse --short HEAD).zip
  ELSE
    SAVE ARTIFACT build/outputs/gerbers.zip gerbers.zip AS LOCAL build/pollysdr-$(git rev-parse --short HEAD).zip
  END

pollysdr-panel-drawing:
  FROM +pollysdr-base

  COPY +pollysdr-panelize/panel.kicad_pcb panel.kicad_pcb
  RUN pcbdraw plot panel.kicad_pcb panel.png
  IF --no-cache [[ $(git status --porcelain=v1 2>/dev/null) ]]
    SAVE ARTIFACT panel.png AS LOCAL build/pollysdr-panel-dirty-$(git rev-parse --short HEAD).png
  ELSE
    SAVE ARTIFACT panel.png AS LOCAL build/pollysdr-panel-$(git rev-parse --short HEAD).png
  END

pollysdr-drawing:
  FROM +pollysdr-base

  RUN pcbdraw plot projects/pollysdr/pollysdr_pcb/pollysdr_pcb.kicad_pcb board.png
  IF --no-cache [[ $(git status --porcelain=v1 2>/dev/null) ]]
    SAVE ARTIFACT board.png AS LOCAL build/pollysdr-dirty-$(git rev-parse --short HEAD).png
  ELSE
    SAVE ARTIFACT board.png AS LOCAL build/pollysdr-$(git rev-parse --short HEAD).png
  END




# About

This 'Game of Life' website was programmed for a
[code competition hosted by IT-Talents.de in 2017](https://www.it-talents.de/foerderung/code-competition/code-competition-09-2017)

# Installing And Running

This project requires no installation.
To start the game, simply open the file called `index.html`, or visit [the online version](https://johannesvollmer.github.io/webgl-of-life/).
in the latest Mozilla Firefox or Google Chrome.

It was tested with Firefox 52.2.0 and Chrome 60.0.3112.78 on Debian Linux x64,
and Chrome 61.0.3163.100 on OSX x64.




## Troubleshooting

1.  It may happen that drawing on the board will not have any visible effect.
    There are several possible reasons for this:
    - Painting with the pattern-mode 'remove' will not add any cells
      to the board, but only remove cells.
      You can change the pattern mode on the left side.
      Alternatively, exit and re-enter the paint mode to reset the pattern-mode.
    - Painting while viewing any other cell-mode than 'life'
      may prevent seeing the result of your brush.
      You can change the cell-mode on the right side.
      Alternatively, exit and re-enter the paint mode to reset the cell-mode.
    - When painting while in play-mode, painting a single cell will
      probably have not effect because it will die instantly.
1.  This website requires JavaScript and WebGL to do anything interesting,
    but a warning will appear if any of that is not available.

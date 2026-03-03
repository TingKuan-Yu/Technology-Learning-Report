# How to convert pdf to jpg

## Reference

- https://stackoverflow.com/questions/43085889/how-to-convert-a-pdf-into-jpg-with-command-line-in-linux

##　pdftoppm

- install tool

  ```bash
  $ sudo apt update && sudo apt install poppler-utils
  ```

- convert

  ```bash
  $ pdftoppm -jpeg -jpegopt quality=100 -r 300 test.pdf img/pg
  ```

- ls

  ```
  $ ls img/
  pg-1.jpg
  ```

  


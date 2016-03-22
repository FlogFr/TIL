=========================
Decrease PDF size in CLI
=========================

.. code-block::
    gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -dNOPAUSE -dQUIET -dBATCH -sOutputFile=output.pdf input.pdf

If the -dPDFSETTINGS=/screen setting is too low quality to suit your needs,
replace it with -dPDFSETTINGS=/ebook for better quality, but slightly larger
pdfs. Delete the setting altogether for the high quality default, which you
can also explicitly call for with -dPDFSETTINGS=/prepress.

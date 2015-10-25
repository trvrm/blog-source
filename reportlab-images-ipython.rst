Reportlab Images in IPython
===========================

:tags: Python, IPython
:category: Software
:author: Trevor
:date: 2015-01-01

With a bit of work we can get IPython to render ReportLab objects
directly to the page as Matplotlib plots.

Huge thanks to github user `deeplook <https://github.com/deeplook>`__,
this is basically a modification of
`this <http://nbviewer.ipython.org/gist/deeplook/5162445>`__ IPython
notebook.

First our imports.

.. code-block::  python

    from reportlab.lib import colors
    from reportlab.graphics import renderPM
    from reportlab.graphics.shapes import Drawing, Rect
    from reportlab.graphics.charts.linecharts import HorizontalLineChart

.. code-block::  python

    from io import BytesIO
    from IPython.core import display

Now we create a hook that causes reportlab drawings to actually be
rendered when we type out its name.

.. code-block::  python

    def display_reportlab_drawing(drawing):
        buff=BytesIO()
        renderPM.drawToFile(drawing,buff,fmt='png',dpi=72)
        data=buff.getvalue()
        ip_img=display.Image(data=data,format='png',embed=True)
        return ip_img._repr_png_()

.. code-block::  python

    png_formatter=get_ipython().display_formatter.formatters['image/png']
    drd=png_formatter.for_type(Drawing,display_reportlab_drawing)

Now that's done, we can start creating ReportLab objects and see them
immediately.

.. code-block::  python

    drawing = Drawing(150,100)
    drawing.add(Rect(0,0,150,100,strokeColor=colors.black,fillColor=colors.antiquewhite))
    drawing



.. image:: reportlab-images-ipython_files/reportlab-images-ipython_8_0.png



.. code-block::  python

    chart=HorizontalLineChart()
.. code-block::  python

    drawing.add(chart)
.. code-block::  python

    drawing



.. image:: reportlab-images-ipython_files/reportlab-images-ipython_11_0.png

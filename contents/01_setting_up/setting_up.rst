Notes on Infrastructure
=======================

For the practical sessions and exercises of this workshop, we have set up a Cloud Server with 32 CPUs and 64Gb of Memory in total (Provider: GWDG_) with a separate user account for each participant. You will have received your . Access to this virtual machine can be obtained using various channels, including a custom terminal client software (such as Terminal on Mac OS X) and Jupyter_, a browser based system for interactive computing. For the following, I will assume access via Jupyter.

.. _GWDG: https://www.gwdg.de

How to Connect to Jupyter
-------------------------

To connect to the Jupyter interface for your virtual machine, open your favourite Browser, go to the `VM Server entry page`_, and click on the Jupyter link indicated for your particular machine. You will be asked to enter the password you have been given for your personal VM.

.. _Jupyter: http://jupyter.org
.. _VM Server entry page: http://195.148.31.27:3000/connect


Basic Usage
-----------
When you first access Jupyter, you will get a file browser view of the ``~/work`` folder on your VM. In the beginning that list will be empty, and will be populated with notebooks and files throughout this workshop. 

To create a new text file, click on *New* and then *Text File*, which opens a text editor within your browser. You can now add content into the file, or edit existing content and save. The filename can be changed by clicking into the Filename on top. You can now go back to your file browser window and update using the button with the two arrows in the upper right corner, and you should see your text file saved under ``~/work`` on your VM.

You can also use Jupyter to open a Terminal within the browser: Click on *New* and then *Terminal*, which will open a terminal window in a separate browser tab. You can enter Unix Bash commands to change directories, view files or execute programs. 

.. note:: To use the terminal, some basic knowledge of Unix is required. For this workshop, you will frequently use tools such as ``pwd`` to view the current directory, ``ls`` to view the contents of the current directory, ``cd`` to change the directory, ``cat`` to output the contents of a file, ``grep`` to search in a text file and other commands.

Finally, you can create new Folders by clicking on *New* and then *Folder*. To rename the new folder, click on the checkbox beside the new folder, and click the *Rename* button on top, which appeared. To change into the new folder, click on it. To move back, click on the parent folder appearing on top of the file browser.

.. admonition:: Exercise

  Create a new folder called ``fun``, and a text file within that folder using Jupyter. Fill the text file with arbitrary content, such as "Hello, World!". Then open a terminal and output the new text file using the ``cat`` command.

Notebooks
---------

Notebook can be loaded for different underlying kernels: bash, python 2, python3 and R. In this tutorial, we will use bash and python 3. Notebooks are useful to document interactive data analysis. It combines code cells with markdown cells. A markdown cell can contain text, math or headings. 

.. admonition:: Exercise

  Create a new bash notebook. Then select in the dropdown list above "Markdown". Type ``# This is a heading`` into the cell, press Shift-Enter and watch. Then type ``This is text with \*italic\* and \*\*bold\*\* letters``. To change the cells, double click into them.

Code cells can be used to write arbitrary code, execute it and get the results printed back into the Notebook.

.. admonition:: Exercise

  A new empty Code cell should have been added to the Notebook in the last step. Click into this code cell and type ``ls``. This should output the current directories and files into the notebook. Into a new cell enter ``NAME="Hello World"`` and in the line below (same cell) ``echo $NAME``.
  
You can use Bash notebooks to perform standard Unix tasks and run programs throughout this workshop. That way, you have always documented what you did.

In Python 3 notebooks you can plot things: Create a new python3 notebook, and use this boilerplate code in the first cell::

  %matplotlib inline
  import matplotlib.pyplot as plt

.. admonition:: Exercise

  Create a simple plot using ``plt.plot([1, 2, 3], [5, 2, 6])``


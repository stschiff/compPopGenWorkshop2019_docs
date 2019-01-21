Setting up
==========

The Computational Infrastructure for this Workshop
--------------------------------------------------

For the practical sessions and exercises of this workshop, we have set up a Cloud Server with 32 CPUs and 64Gb of Memory in total with a separate user account for each participant. As participant, you will have received your own username and password for accessing the server. The simplest way to access the server is via Jupyter:

`https://c107-224.cloud.gwdg.de/hub/login`_

.. _https://c107-224.cloud.gwdg.de/hub/login: https://c107-224.cloud.gwdg.de/hub/login

where you can enter your username and password and are then logged into Jupyter. For reference, you find documentation for the Jupyter interface `here <https://jupyter.org/documentation>`_

More expert users can also access the server via custom terminal client software (such as ``Terminal`` on Mac OS X, or ``putty`` on Windows), via::

    ssh <USERNAME>@c107-224.cloud.gwdg.de

Basic Usage
-----------
When you first access Jupyter, you will get a file browser view of your home directory on the server. In the beginning, your home directory will be empty, and will be populated with notebooks and files throughout this workshop.

To create a new text file, click on *New* (in the upper right corner) and then *Text File*, which opens a text editor within your browser. You can now add content into the file, or edit existing content and save. The filename can be changed by clicking into the Filename on top. You can now go back to your file browser window and update using the button with the two arrows in the upper right corner, and you should see your text file saved in your home directory.

You can also use Jupyter to open a Terminal within the browser: Click on *New* and then *Terminal*, which will open a terminal window in a separate browser tab. You can enter Unix Bash commands to change directories, view files or execute programs (as we will learn below). 

Finally, you can create new Folders by clicking on *New* and then *Folder*. To rename the new folder, click on the checkbox beside the new folder, and click the *Rename* button on top, which appeared. To change into the new folder, click on it. To move back, click on the parent folder appearing on top of the file browser.

.. admonition:: Exercise

  Create a new folder called ``hello``, and a text file within that folder using Jupyter. Name that text file ``hello.txt`` and fill it with arbitrary content, such as "Hello, World!". Then open a terminal and output the contents of the new text file typing ``cat hello/hello.txt`` followed by ENTER.

.. note::
   
   While the Jupyter terminal and Jupyter Text Files are different ways to interact with the server, both access the same file system. So files created with the Text Editor are saved in your home directory, and can be accessed via the terminal, and vice versa: Files created via the Terminal can be accessed via the Text Editor, by simpling clicking on them in the Jupyter File Browser.

Notebooks
---------

Notebook can be loaded for different underlying kernels: bash, python and R. Notebooks are useful to document interactive data analysis. It combines code cells with markdown cells. A markdown cell can contain text, math or headings. 

.. admonition:: Exercise

  Create a new bash notebook. Then select in the dropdown list above "Markdown". Type ``# Bash Exercises`` into the cell, press Shift-Enter and watch. Then type ``This is text with \*italic\* and \*\*bold\*\* letters``. Then again type Shift-ENTER and watch.
  
.. hint::
   
   To change the cells, double click into them.

Code cells can be used to write arbitrary code, execute it and get the results printed back into the Notebook.

.. admonition:: Exercise

  A new empty Code cell should have been added to the Notebook in the last step. Click into this code cell and type ``ls``. This should output the current directories and files into the notebook. Into a new cell enter ``NAME="Hello World"`` and in the line below (same cell) ``echo $NAME``. Then again Shift-ENTER.
  
You can use Bash notebooks to perform standard Unix tasks and run programs throughout this workshop. That way, you have always documented what you did.

In Python 3 notebooks you can plot things: Create a new python3 notebook, and use this boilerplate code in the first cell::

  %matplotlib inline
  import matplotlib.pyplot as plt

Then plot something:

.. admonition:: Exercise

  Create a simple plot using ``plt.plot([1, 2, 3], [5, 2, 6])``

Working with Bash
-----------------

Bash denotes both a scripting language and the interactive system that is used with Terminals (and Bash notebooks). In the following, we will learn some basic use cases and commands.

The most important commands in a Unix shell are ``ls``, ``cd`` and ``mkdir``. In general, all commands in a Unix shell are entered by typing the command and then typing ENTER.

.. admonition:: Exercise
 
  Open a Terminal and execute the ``ls`` command by typing ``ls``. It should list you the contents of your home directory. Then run ``mkdir testdir`` to create a new directory. Then type ``cd testdir`` to change into that new directory.

Let's learn some more commands. Above you have already used ``cat`` to output the contents of a file to the screen. Another important command is ``echo``, which also prints stuff to the screen, but not from a file but from a string that you give it. For example:

.. admonition:: Exercise

  try the command ``echo "Hello, how are you?"`` in your terminal.


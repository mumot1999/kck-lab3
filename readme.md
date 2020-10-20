```
jupytext *.ipynb --to md
jupytext gardients.md --to ipynb

ipython nbconvert --to html *.ipynb
 ```
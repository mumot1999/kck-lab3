---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.6.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

```python
import matplotlib
import matplotlib.pyplot as plt
from matplotlib import rc
import numpy as np
from matplotlib import colors
```

Funkcje do zaimplementowania:

```python
#konwerter: nie trzeba implementować samemu, można wykorzystać funkcję z bilbioteki
def hsv2rgb(h, s, v):
    return colors.hsv_to_rgb((h, s, v))

# poniżej znajdują się funkcje modelujące kolejne gradienty z zadania.
# v to pozycja na osi ox: v jest od 0 do 1. Zewnetrzna funkcja wywołuje te metody podając
# różne v i oczekując trójki RGB bądź HSV reprezentującej kolor. Np. (0,0,0) w RGB to kolor czarny. 
# Należy uwikłać v w funkcję modelującą kolor. W tym celu dla kolejnych gradientów trzeba przyjąć 
# sobie jakieś punkty charakterystyczne,
# np. widzimy, że po lewej stronie (dla v = 0) powinien być kolor zielony a w środku niebieski (dla v = 0.5),
# a wszystkie punkty pomiędzy należy interpolować liniowo (proporcjonalnie). 

def lin(start, end, inc=1, move=0):
    diff = (np.array(end) - np.array(start))

    def get_color(x):
        return tuple(start + (diff * x * inc) + move)

    return get_color

def get_color_function(map):
    def fun(v):
        for previous , current in zip([None, *list(map.items())], list(map.items())):
            k, fun = current
            k = k/100

            diff = 0

            if previous:
                diff = previous[0]/100
                increment = 1/ (k-diff)
            else:
                increment = 1/k

            if k >= v:
                return fun((v-diff) * increment)

    return fun

def gradient_rgb_bw(v):
    return get_color_function({
            100: lin((0,0,0), (1,1,1)),
        })(v)

def gradient_rgb_gbr(v):
    return get_color_function({
        50: lin((0,1,0), (0,0,1)),
        100: lin((0,0,1), (1,0,0))
    })(v)

def gradient_rgb_gbr_full(v):
    return get_color_function({
        25: lin((0,1,0), (0,1,1)),
        50: lin((0,1,1), (0,0,1)),
        75: lin((0,0,1), (1,0,1)),
        100: lin((1,0,1), (1,0,0))
    })(v)


def gradient_rgb_wb_custom(v):
    return get_color_function({
        40: lin((0,1,0.5), (1,1,1)),
        50: lin((1,1,1), (1,0.5,0)),
        90: lin((1,0.5,0), (1,0,0)),
        100: lin((1,0,0), (0,1,0))
    })(v)


def gradient_hsv_bw(v):
    #TODO
    return hsv2rgb(1, 0, v)


def gradient_hsv_gbr(v):
    return hsv2rgb((0.35 + v)*0.76, 1, 1)

def gradient_hsv_unknown(v):
    #TODO
    return hsv2rgb(0.3 - v * 0.3, 0.5, 1)


def gradient_hsv_custom(v):
    #TODO
    return hsv2rgb(v, 1, 1)
```

```python
def plot_color_gradients(gradients, names):
    # For pretty latex fonts (commented out, because it does not work on some machines)
    #rc('text', usetex=True) 
    #rc('font', family='serif', serif=['Times'], size=10)
    rc('legend', fontsize=10)

    column_width_pt = 400         # Show in latex using \the\linewidth
    pt_per_inch = 72
    size = column_width_pt / pt_per_inch

    fig, axes = plt.subplots(nrows=len(gradients), sharex=True, figsize=(size, 0.75 * size))
    fig.subplots_adjust(top=1.00, bottom=0.05, left=0.25, right=0.95)


    for ax, gradient, name in zip(axes, gradients, names):
        # Create image with two lines and draw gradient on it
        img = np.zeros((2, 1024, 3))
        for i, v in enumerate(np.linspace(0, 1, 1024)):
            img[:, i] = gradient(v)

        im = ax.imshow(img, aspect='auto')
        im.set_extent([0, 1, 0, 1])
        ax.yaxis.set_visible(False)

        pos = list(ax.get_position().bounds)
        x_text = pos[0] - 0.25
        y_text = pos[1] + pos[3]/2.
        fig.text(x_text, y_text, name, va='center', ha='left', fontsize=10)

    fig.savefig('my-gradients.pdf')
```

```python
def toname(g):
    return g.__name__.replace('gradient_', '').replace('_', '-').upper()
    
gradients = (gradient_rgb_bw, gradient_rgb_gbr, gradient_rgb_gbr_full, gradient_rgb_wb_custom,
                 gradient_hsv_bw, gradient_hsv_gbr, gradient_hsv_unknown, gradient_hsv_custom)

plot_color_gradients(gradients, [toname(g) for g in gradients])
```

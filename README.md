# FdF — Wireframe Renderer

**FdF** ("Fil de Fer", *wire frame* in French) reads a `.fdf` heightmap and draws
it as a 3D **isometric wireframe** using the **MiniLibX** graphics library. A
classic 42 introduction to coordinate transforms and line rasterization.

> 42 project.

---

## Table of contents

- [What it does](#what-it-does)
- [Build](#build)
- [Run](#run)
- [Controls](#controls)
- [`.fdf` map format](#fdf-map-format)
- [How it works](#how-it-works)
- [Project structure](#project-structure)
- 
---

## What it does

- Parses a `.fdf` file of elevation values into a list of 3D points.
- Projects the grid into an **isometric** view and auto-centers it in the window.
- Draws the wireframe (each point linked to its right and bottom neighbor) with
  **Bresenham's** line algorithm.
- Colors each line by **elevation** via a built-in height gradient
  (`get_color`), from purple/blue for negative heights to green for high ones.
- Lets you pan the model with the keyboard in real time.

---

## Build

The repository vendors its own `libft` and `minilibx-linux`; `make` builds both,
then the project.

```bash
make
```

Produces the `fdf` executable. System dependencies (Linux / X11): a C compiler,
`make`, and the X11 dev libraries — the binary links `-lmlx -lXext -lX11 -lm`
(and `-lft`).

Targets: `make`, `make clean`, `make fclean`, `make re`.

---

## Run

```bash
./fdf test_maps/42.fdf
```

The program takes exactly one argument, whose name must contain `.fdf`. Many
sample maps are provided in `test_maps/` (e.g. `elem.fdf`, `42.fdf`,
`mars.fdf`, `pyramide.fdf`). A 2000×2000 window titled *Fdf* opens.

---

## Controls

| Key | Action |
|-----|--------|
| `W` / `S` | Pan the view up / down |
| `A` / `D` | Pan the view left / right |
| `Esc` | Quit |

Closing the window also exits. (There is no zoom or rotation control in this
version — only translation.)

---

## `.fdf` map format

A plain-text grid of space-separated integers; each value is the **Z (height)**
of a point, its row/column giving X and Y.

```
0  0  0  0
0  1  2  0
0  0  0  0
```

Some maps in `test_maps/` append a per-point color as `Z,0xRRGGBB`
(e.g. `20,0xFF0000`). **This program ignores that color part** — heights are
read with `ft_atoi`, which stops at the comma, and line colors come from the
elevation gradient instead.

---

## How it works

1. **Parse** (`main.c`, `get_point_pos.c`) — read the file line by line with
   `get_next_line`, split each row on spaces, and build a linked list of
   `t_point {x, y, z}`. The map width is the number of columns.
2. **Project & center** (`center_pos.c`) — apply the isometric transform to
   every point (rotation by ~26.155°, scaled by window-width / map-width, with
   Z lifting the point), compute the bounding box, and offset everything so the
   shape is centered in the window.
3. **Draw** (`FdF.c`, `draw.c`) — for each point, draw a line to its right
   neighbor (unless it's at the end of a row) and to the point one row below;
   `draw_line` is a Bresenham rasterizer writing pixels into an MLX image, which
   is then pushed to the window.
4. **Interact** (`key_press`) — on a movement key, every point is shifted, the
   image is rebuilt and redrawn.

---

## Project structure

```
Fdf/
├── Makefile
├── fdf.h               # structs (t_point, t_mlx, t_draw, …) and prototypes
├── main.c              # arg check, file reading, map building
├── get_point_pos.c     # parse rows/elements into the point list
├── center_pos.c        # isometric projection + auto-centering
├── FdF.c               # window/loop, neighbor linking, key handling
├── draw.c              # height-based color + Bresenham line drawing
├── utils.c             # helpers (ft_strstr, width count, cleanup)
├── exit.c              # error paths and resource freeing
├── test_maps/          # many sample .fdf maps
├── libft/              # vendored
└── minilibx-linux/     # vendored
```

# B-System for Houdini

B-System is a instance editing plugin for [SideFX Houdini](https://www.sidefx.com/products/houdini/), you can get it on [Gumroad](https://ae43ae43.gumroad.com/l/bahri).

## Installation

B-System uses the [package](https://www.sidefx.com/docs/houdini/ref/plugins.html) format for Houdini plugins. It supports installing from a package archive:

1. Download `.zip` version from [Gumroad](https://ae43ae43.gumroad.com/l/bahri).
2. Run Houdini and open the **Package Browser** pane, found under `Inspectors > Package Browser`.
3. Go to `File > Install Package Archive...`.
	- Set **Archive** to the path of the `bsystem<version>.zip` file.
 	- Set **Install to** to the `$HOUDINI_PACKAGE_DIR`, this defaults to `~/Documents/houdini<version>/packages` on Windows, and `~/houdini<version>` on Linux and MacOS.

> NOTE: If you install to another directory you need to set the `BSYSTEM` variable in the `bsystem<version>.json` package file to the correct install directory.

> The `BSYSTEM` environment variable is required by the plugin, don't set the `"hpath"` directly.

```json
{
  "enable": true,
  "env": [
    {
      "BSYSTEM": "$HOUDINI_USER_PREF_DIR/bsystem<version>"
    }
  ],
  "path": "$BSYSTEM"
}
```

# Documentation

For details on **nodes** and node **parameters** see the node help cards in Houdini.

## Quickstart

Use the `B-System Configure Basic` for a quick basic setup.

![B-System Configure Basic](https://github.com/aeaeaeaeaeae/bsystem/blob/main/imgs/B-System%20Configure%20Basic.gif)

## Toturials

+ [Viewport, handles and editing](https://youtu.be/edli7ctcxrU)
+ [Node usage, palette and assemblies](https://youtu.be/JosBn27ilNQ)
+ [Geometry, attributes and groups](https://youtu.be/nW44NoTxzSE)
+ [Examples and use cases](https://youtu.be/M6UfgjYinjk)

## B-System geometry

The geometry used in the B-System Edit SOP follows a simple convention that defines the instances and their transform, the hierarchy between them and the direction of this hierarchy. Be aware that the hierarchy can't contain loops, loops will be broken up when merged into a B-System Edit SOP.

### Attribute and groups

|           | vex             | class | type            | description                       |
| --------- | --------------- | ----- | --------------- | --------------------------------- |
| name      | `s@name`        | point | string          | Instance name                     |
| transform | `3@transform`   | point | rotation matrix | Instance rotation and scale       |
| roots     | `i@group_roots` | point | group           | Origin and direction of hierarchy |
| assembly  | `s@assembly`    | point | string          | Defines assembly                  |

### Instance name `s@name`

The `s@name` point attributes stores the instance name, this works the same way as named instancing works on the `Copy to Points SOP`. Instances must be packed primitives and have a corresponding `s@name` attribute. All packed primitive types are supported.

### Instance transform `3@transform`

The `3@transform` point attribute stores the instance rotation and scale. The rotation matrix is in world space. It works the same way as **KineFX** bone transforms.

Note that `p@orient`, `@N`, `@up`, `@scale` and `@pscale` is not used for setting the rotation and scale, these attributes are ignored. Check out the vex functions [`maketransform()`](https://www.sidefx.com/docs/houdini/vex/functions/maketransform.html) and [`scale()`](https://www.sidefx.com/docs/houdini/vex/functions/scale.html) for ways to quickly convert these attributes to a rotation matrix.

```c
// N and up to Matrix3
3@transform = maketransform(@N, @up);
scale(3@transform, set(@pscale, @pscale, @pscale));
```

```c
// Quaternion to Matrix3
3@transform = qconvert(p@orient);
scale(3@transform, set(@pscale, @pscale, @pscale));
```

### Roots group

Instances on a B-System are connected in a branching hierarchy, the `roots` point group defines the origin and direction of this hierarchy. A set of connected instances can only have a single root point. Multiple connected sets will have a root for each set. Single unconnected instances are also roots.

Using a point to define the direction of the hierarchy is simpler than using vertex order, since vertex order is tricky to visualize, edit and debug in Houdini.

### Assembly name s@assembly

Assemblies are labeled sets of instances, they are defined by the `s@assembly` point attribute. An assembly does not have to be fully connected, from any combination of connected and unconnected instances. Apart from the `s@assembly` attribute the definition of an assembly is identical to regular bsystem geometry.

## Hotkeys

See the viewer state info panel for hotkeys.

> For the Houdini `20.0` version certain keys and inputs remain unavailable for python viewer states. Among others these includes the `WERT` keys, the `ctrl` key while dragging, and `RMB` if using a `Context Menu`. For this reason certain operations map to less intuitive keys such as `a` for rotation instead of `r`, and `shift` to snap instead of `ctrl`. This will change as soon as SideFX frees the keys.

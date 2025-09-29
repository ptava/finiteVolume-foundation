# FiniteVolume library
Using: [OpenFOAM-dev](https://openfoam.org)
Build: dev-bc70899b1df1
### Overview
The changes made addresses an error encountered at the first topoChange when using `timeVaryingMappedFixedValue` boundary condition

#### What seemed to be broken
During the first topo change (refine) the mesh calls:
```
fvMesh::mapFields -> MapGeometricFields -> timeVaryingMappedFixedValueFvPatchField::map/reset
```

For identity mappings (i.e., faces unchanged), the current code can end up doing

```
Field<Type>::reset( self )
```
which aborts with error: attempted to assign to self

#### Fix in BC
Add one-line guard at the start of both `map` and `reset` functions in `timeVaryingMappedFixedValueFvPatchField.C`:

```cpp
if (this == &ptf) return;
```

When topo-change requests an identity map (old patch field == new patch field), the guard turns the operation into a no-op (i.e., "nothing changed").

This prevents the base implementation from reaching Field::reset(self), which is what trips the fatal error.


# [RFC] Update valueBoundsOpInterface to compute bounds of dynamic stride/offset sizes

### Background

The `valueBoundsOpInterface` is a great design in mlir, widely used for analysis and transformation passes. However, its inability to compute dynamic stride or offset sizes presents a limitation. 

Given the prominence of `stride` and `offset` in the `mlir::MemRef` dialect, addressing this deficiency is essential. We propose extending the `valueBoundsOpInterface` to incorporate dynamic stride and offset calculations.

### Extend Interface Methods
```c++
void populateBoundsForOffsetValue(Value value, ValueBoundsConstraintSet &cstr)

void populateBoundsForStridedValueDim(Value value, int64_t dim, ValueBoundsConstraintSet &cstr)
```

The `ValueDim` or `ValueDimList` in `ValueBoundsConstraintSet` should be deleted, because it can not save enougn information to cover `IndexValue`, `ShapedValueDim`, `StridedValueDim` and `OffsetValue`. Then add `PopulateMode` enum  and `ColumnNode` class to solve the probelm.

The `PopulateMode` represents the target of the function to compute is `IndexValue`, `ShapedValue`, `StridedValue` or `OffsetValue`. The `ColumnNode` containing the `value`, `dim` and `PopulateMode` can cover the all information 

The demo code likes belows: 

```c++
enum PopulateMode {
  INDEX = 0,
  SHAPE,
  STRIDE,
  OFFSET,
};

struct ColumnNode {
  Value value;
  int64_t dim;
  PopulateMode pMode;
};
```

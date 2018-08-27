# math.Abs()
math.Abs()のコードリーディング。

https://github.com/golang/go

## src/math/abs.go
関数本体。まずはFloat64bits()とFloat64frombits()を探す。

```go
// Abs returns the absolute value of x.
//
// Special cases are:
//	Abs(±Inf) = +Inf
//	Abs(NaN) = NaN
func Abs(x float64) float64 {
	return Float64frombits(Float64bits(x) &^ (1 << 63))
}
```

## src/math/unsafe.go
float値をIEEE754形式の符号なし64bit整数で返す。
複数個あったがmathパッケージなのでおそらくこれ。
unsafe.Pointer()は問答無用でvoid*に変換するやつかしら？

```go
// Float64bits returns the IEEE 754 binary representation of f.
func Float64bits(f float64) uint64 { return *(*uint64)(unsafe.Pointer(&f)) }
```

Float64frombitsはIEE754形式の符号なし64bit整数をfloat64に戻す。

```go
// Float64frombits returns the floating point number corresponding
// the IEEE 754 binary representation b.
func Float64frombits(b uint64) float64 { return *(*float64)(unsafe.Pointer(&b)) }
```

## src/math/abs.go
ここで再び元の関数に戻る。

1. xをFloat64bits()で64bit整数に
2. `1 << 63`で一番左ビットのみ立った64bit整数
3. `&^`はgolangの[bit clear](https://golang.org/ref/spec#Arithmetic_operators)演算子
4. `1 &^ 2`なので一番左のビットがクリアされることになる
5. IEEE 754によると[最上位ビットは符号](https://ja.wikipedia.org/wiki/IEEE_754)なので符号ビットがクリアされることになる
6. 最後にFloat64frombits()でfloat64に戻して終了

```go
return Float64frombits(Float64bits(x) &^ (1 << 63))
```

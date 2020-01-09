# vue-scroll

タイムラインUI実現のための思考

ズーマブルUIには下記の３つの方法がある。

- centerとzoomを持ち、左端と右端を計算してdomainに与える
- 単にstartとendを持つ
- start + zoomからendを計算しdomainに与える

どの場合でも、scaleはcomputedから供給される関数とする。

## License

MIT
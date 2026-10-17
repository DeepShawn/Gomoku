# 五子棋 · Rapfi AI

一个离线可玩的网页五子棋，内置 [Rapfi](https://github.com/dhbloo/rapfi) 引擎（WebAssembly + NNUE），打开即战，无需联网。

在线地址：<https://deepshawn.github.io/Gomoku/>

## 玩法

- 15×15 棋盘，黑棋先手。
- 黑棋双活三、双四、长连（六子及以上）为禁手；白棋无禁手。
- 禁手点由 Rapfi 的连珠规则引擎判定，落子前实时校验并拦截提示。
- 可选执黑先手或执白后手；支持悔棋、新对局、禁手点标记。
- 三档难度对应引擎思考时间：简单约 0.2 秒（限深）、普通约 0.9 秒、困难约 2 秒。

## 关于引擎

- 引擎：[Rapfi](https://github.com/dhbloo/rapfi)，α-β 搜索 + mix9svq NNUE 评估，编译为单文件 WebAssembly。
- 权重：[rapfi-networks](https://github.com/dhbloo/rapfi-networks) 的连珠 15×15 黑白 NNUE 权重（CC0）。
- 页面为单文件 `index.html`，WASM、配置与权重全部内联，无任何外部请求；首次打开需解压约 20MB 权重，通常 2–3 秒可用。

## 本地运行

直接双击 `index.html`，或用任意静态服务器托管：

```bash
python3 -m http.server 8000
# 打开 http://localhost:8000/
```

## 构建说明

想自行编译引擎并重新组装页面：

```bash
git clone --depth 1 https://github.com/dhbloo/rapfi.git
cd rapfi && git submodule update --init Networks
mkdir -p Rapfi/build/wasm && cd Rapfi/build/wasm
emcmake cmake ../.. -DCMAKE_BUILD_TYPE=Release -DNO_COMMAND_MODULES=ON \
  -DNO_MULTI_THREADING=ON -DUSE_WASM_SIMD=ON
emmake cmake --build . -j2
```

WASM 构建需注意：使用 `-s TOTAL_STACK=2MB`、`-s SINGLE_FILE=1`，导出函数不要包含 `_main`（Emscripten 会以 `__main_argc_argv` 导出 C++ 入口），并把 `Networks/wasm_preloads.txt` 清空以避免预载权重。

页面通过 Worker 内的 Gomocup/Piskvork 协议驱动引擎：

```
START 15
INFO RULE 4
BOARD x,y,color ... DONE
```

## 许可证

页面内分发的 Rapfi 引擎遵循 [GNU GPL v3](https://www.gnu.org/licenses/gpl-3.0.html)，完整许可证文本已内嵌在页面的「规则与引擎」面板中；引擎源码见 <https://github.com/dhbloo/rapfi>。NNUE 权重以 CC0 发布。

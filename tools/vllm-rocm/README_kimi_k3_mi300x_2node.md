# Kimi-K3 on 2x 8x MI300X (TP8 x PP2)

Serving config validated 2026-08-17. Requires the k3 branch fixes:
padding-mask fix (66ee2226e) and gating-mask no-op (618c30461).

Cudagraph mode MUST be FULL_DECODE_ONLY: the piecewise-compiled prefill
path corrupts output under pipeline parallelism (upstream bug, eager
prefill avoids it), while full-graph decode capture is correct.

    VLLM_ROCM_USE_AITER=1 VLLM_ROCM_USE_AITER_MOE=1     vllm serve moonshotai/Kimi-K3       --tensor-parallel-size 8 --pipeline-parallel-size 2       --distributed-executor-backend ray       --compilation-config "{\"cudagraph_mode\": \"FULL_DECODE_ONLY\"}"       --gpu-memory-utilization 0.97 --max-num-seqs 128       --served-model-name kimi-k3 --reasoning-parser kimi_k3       --tool-call-parser kimi_k3 --enable-auto-tool-choice       --trust-remote-code

Measured (MI300X, 16 GPUs): 36.4 tok/s single-stream,
168.5 tok/s @ 8 concurrent, 498.3 tok/s @ 32 concurrent.
Client sampling: temperature 1.0, top_p 0.95, thinking_effort low.

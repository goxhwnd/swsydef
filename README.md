def solution(visible, hidden, k):
    n, m = len(visible), len(visible[0])
    same_color = ((n + m) % 2 == 0)      # 시작·끝 색 동일?

    INF = 1010                           # 100보다 큰 값
    BEST = -10**9

    # 미리 2차원 → 1차원으로 복사(속도용)
    vis = [row[:] for row in visible]
    hid = [row[:] for row in hidden]

    for rmask in range(1 << n):          # 1. 행 패턴 열거
        row_cost = k * bin(rmask).count("1")

        # 열별 미리 계산
        col0 = [0] * m; col1 = [0] * m
        min0 = [INF] * m; min1 = [INF] * m

        for i in range(n):
            ri = (rmask >> i) & 1
            for j in range(m):
                v0 = vis[i][j] if ri == 0 else hid[i][j]
                v1 = hid[i][j] if ri == 0 else vis[i][j]

                col0[j] += v0
                col1[j] += v1

                if same_color and ((i + j) & 1):   # 백 셀만 추적
                    min0[j] = min(min0[j], v0)
                    min1[j] = min(min1[j], v1)

        if not same_color:               ## 2-A. 끝점 색 다름
            dp = -row_cost
            for j in range(m):
                dp = max(dp + col0[j], dp + col1[j] - k)
            BEST = max(BEST, dp)
        else:                            ## 2-B. 끝점 색 같음
            S = 102                      # 1..100, 101=∞
            dp = [-10**9] * S
            dp[101] = -row_cost          # 아직 백 셀이 없음

            for j in range(m):
                ndp = [-10**9] * S
                for mv, val in enumerate(dp):
                    if val < -1e8: continue

                    # 그대로
                    nm = min(mv, min0[j]) if mv != 101 else min0[j]
                    if nm == INF: nm = 101
                    ndp[nm] = max(ndp[nm], val + col0[j])

                    # 뒤집음
                    nm = min(mv, min1[j]) if mv != 101 else min1[j]
                    if nm == INF: nm = 101
                    ndp[nm] = max(ndp[nm], val + col1[j] - k)
                dp = ndp

            for mv, val in enumerate(dp):
                if val < -1e8: continue
                BEST = max(BEST, val if mv == 101 else val - mv)

    return BEST

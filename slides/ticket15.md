## Умножение плотных матриц

Умножение плотных матриц можно реализовать втупую с очевидным memory bound, а можно следующим образом: перемножать куски например 16×16, добавлять их в локальную память, а потом доставать

const uint loc_i = get_local_id(0);
    const uint loc_j = get_local_id(1);
    const uint i = get_global_id(0);
    const uint j = get_global_id(1);

    __local float tile_a[GROUP_SIZE_Y][GROUP_SIZE_X];
    __local float tile_b[GROUP_SIZE_Y][GROUP_SIZE_X + 1];

    c[j * w + i] = 0.0f;

    for (uint tile_k = 0; tile_k < k; tile_k += GROUP_SIZE_X) {
        const uint col_a = tile_k + loc_i;
        const uint row_b = tile_k / GROUP_SIZE_X * GROUP_SIZE_Y + loc_j;

        if (j < h && col_a < k) {
            tile_a[loc_j][loc_i] = a[j * k + col_a];
        } else {
            tile_a[loc_j][loc_i] = 0.0f;
        }

        if (i < w && row_b < k) {
            tile_b[loc_j][loc_i] = b[row_b * w + i];
        } else {
            tile_b[loc_j][loc_i] = 0.0f;
        }

        barrier(CLK_LOCAL_MEM_FENCE);

        for (uint acc_idx = 0; acc_idx < GROUP_SIZE_X; ++acc_idx) {
            c[j * w + i] += tile_a[loc_j][acc_idx] * tile_b[acc_idx][loc_i];
        }

        barrier(CLK_LOCAL_MEM_FENCE);
    }

  Так же можно ускорить это всё дело с помощью wwma, тензорных ядер, квантования и тд

## Массив префиксных сумм по массиву чисел

Нужно 2 кернела на каждый шаг
1 кернел из массива размером 2^k делает массив размером 2^(k-1), то есть редуцирует

const uint index = get_global_id(0);

    if (index >= (n + 1) / 2)
        return;

    const uint first = pow2_sum[index * 2];
    const uint second = index * 2 + 1 < n ? pow2_sum[index * 2 + 1] : 0;

    next_pow2_sum[index] = first + second;

2 кернел аккумулирует префиксную сумму: отбрасывает чётные блоки и прибавляет к сумме префиксную сумму на нечётном блоке

const uint index = get_global_id(0);

    if (index >= n || (((index + 1) >> pow2) & 1) == 0)
        return;

    prefix_sum_accum[index] += pow2_sum[((index + 1) >> pow2) - 1];


Потенциал ускорения - делать редукцию не в 2 раза, а в 2^k
ке

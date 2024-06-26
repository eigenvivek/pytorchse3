pytorchse3
================

**⚠️ PyTorch3D v0.7.6 [improved the numerical stability](https://github.com/facebookresearch/pytorch3d/commit/292acc71a33bf389225ef02af237dd82a8319f59) of their `so3_log_map`/`so3_exp_map`, addressing the main issues fixed by this package. Therefore, you should install PyTorch3D instead for SO(3)/SE(3) transformations.**

## Install

``` sh
pip install pytorchse3
```

## How to use

``` python
import torch

from pytorchse3.se3 import se3_exp_map, se3_log_map
```

Here are two transformation matrices for which `PyTorch3D` recovers the
wrong log map (see [this
issue](https://github.com/facebookresearch/pytorch3d/issues/1609?notification_referrer_id=NT_kwDOAcYOvLM3MzY1NTAxMTY0OjI5NzU3MTE2#issuecomment-1839450529)).

``` python
T = torch.Tensor(
    [
        [
            [-0.7384057045, 0.3333132863, -0.5862244964, 0.0000000000],
            [0.3520625532, -0.5508944392, -0.7566816807, 0.0000000000],
            [-0.5751599669, -0.7651259303, 0.2894364297, 0.0000000000],
            [-0.1840534210, -0.1836946011, 0.9952554703, 1.0000000000],
        ],
        [
            [-0.7400283217, 0.5210028887, -0.4253400862, 0.0000000000],
            [0.5329059958, 0.0683888718, -0.8434065580, 0.0000000000],
            [-0.4103286564, -0.8508108258, -0.3282552958, 0.0000000000],
            [-0.1197679043, 0.1799146235, 0.5538908839, 1.0000000000],
        ],
    ],
).transpose(-1, -2)
```

`pytorchse3` computes the correct log map.

``` python
log_T_vee = se3_log_map(T)
log_T_vee
```

    tensor([[ 1.1319,  1.4831, -2.5131, -0.8503, -0.1170,  0.7346],
            [ 1.1288,  2.2886, -1.8147, -0.8812,  0.0367, -0.1004]])

Exponentiating the log map recovers the original transformation matrix
with 1e-4 absolute error.

``` python
eq_T = se3_exp_map(log_T_vee)
assert torch.allclose(T, eq_T, atol=1e-4)
```

``` python
T - eq_T
```

    tensor([[[-9.2983e-06, -2.3842e-07,  1.1504e-05,  2.9802e-08],
             [-5.1558e-06,  8.5235e-06, -8.6427e-06, -2.9802e-08],
             [ 8.6427e-06, -6.4373e-06,  4.4703e-07,  0.0000e+00],
             [ 0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00]],

            [[ 8.0466e-06,  1.6212e-05,  6.0201e-06, -3.7253e-08],
             [ 4.5896e-06,  8.6352e-06,  3.3975e-06,  2.9802e-08],
             [-8.5831e-06,  1.0610e-05, -1.6809e-05,  0.0000e+00],
             [ 0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00]]])

## References

- `pytorchse3` implements log/exp maps defined in Section 2 and 3 of
  [Ethan Eade’s tutorial](https://ethaneade.com/lie.pdf)
- Our numerically stable
  [`so3_log_map`](https://vivekg.dev/pytorchse3/so3.html#so3_log_map) is
  a PyTorch port of
  [`pytransform3d`](https://github.com/dfki-ric/pytransform3d/blob/c45e817c4a7960108afe9f5259542c8376c0e89a/pytransform3d/rotations/_conversions.py#L1719-L1787)
- Taylor expansions for some coefficients in
  [`se3_log_map`](https://vivekg.dev/pytorchse3/se3.html#se3_log_map)
  are taken from
  [`H2-Mapping`](https://github.com/SYSU-STAR/H2-Mapping/blob/11b8ab15f3302ccb2b4b3d2b30f76d86dcfcde2c/mapping/src/se3pose.py#L89-L118)

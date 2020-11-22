---
title: DAPLINK资料篇1
categories: STM32 DAPLINK
tags: STM32 DAPLINK
description: 资料
---
# IDCODE

<table>
<thead>
<tr>
<th style="text-align: center" rowspan="2">分组</th>
<th style="text-align: center" rowspan="2">系列</th>
<th style="text-align: center" rowspan="2">型号</th>
<th style="text-align: center" colspan="2">IDCODE</th>
<th style="text-align: center" rowspan="2">容量地址</th>
<th style="text-align: center" rowspan="2">UNID</th>
</tr>
<tr>
<th style="text-align: center">地址</th>
<th style="text-align: center">DEV_ID</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" rowspan="4">STM32G0</td>
<td style="text-align: center" rowspan="2">STM32G0x0</td>
<td style="text-align: center">STM32G070xx</td>
<td style="text-align: center" rowspan="2">0x4001 5800</td>
<td style="text-align: center">0x460</td>
<td style="text-align: center" rowspan="2">0x1FFF 75E0</td>
</tr>
<tr>
<td style="text-align: center">STM32G030xx</td>
<td style="text-align: center">0x466</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32G0x1</td>
<td style="text-align: center">STM32G071xx<br/>STM32G081xx</td>
<td style="text-align: center" rowspan="2">0x4001 5800</td>
<td style="text-align: center">0x460</td>
<td style="text-align: center" rowspan="2">0x1FFF 75E0</td>
<td style="text-align: center" rowspan="2">0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center">STM32G031xx<br/>STM32G041xx</td>
<td style="text-align: center">0x466</td>
</tr>
<tr>
<td style="text-align: center" rowspan="3">STM32G4</td>
<td style="text-align: center" rowspan="3">STM32G4</td>
<td style="text-align: center">STM32G431<br/>STM32G441</td>
<td style="text-align: center" rowspan="3">0xE004 2000</td>
<td style="text-align: center">0x468</td>
<td style="text-align: center" rowspan="3">0x1FFF 75E0</td>
<td style="text-align: center" rowspan="3">0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center">STM32G47x<br/>STM32G48x</td>
<td style="text-align: center"> 0x469</td>
</tr>
<tr>
<td style="text-align: center">STM32G491<br/>STM32G4A1</td>
<td style="text-align: center"> 0x479</td>
</tr>
<tr>
<td style="text-align: center" rowspan="10">STM32F0</td>
<td style="text-align: center" rowspan="5">STM32F0x0</td>
<td style="text-align: center">STM32F030x4<br/>STM32F030x6</td>
<td style="text-align: center" rowspan="5">0x4001 5800</td>
<td style="text-align: center">0x444</td>
<td style="text-align: center" rowspan="5">0x1FFF F7CC</td>
<td style="text-align: center" rowspan="5"></td>
</tr>
<tr>
<td style="text-align: center">STM32F070x6</td>
<td style="text-align: center">0x445</td>
</tr>
<tr>
<td style="text-align: center">STM32F030x8</td>
<td style="text-align: center">0x440</td>
</tr>
<tr>
<td style="text-align: center">STM32F070xB</td>
<td style="text-align: center">0x448</td>
</tr>
<tr>
<td style="text-align: center">STM32F030xC</td>
<td style="text-align: center">0x442</td>
</tr>
<tr>
<td style="text-align: center" rowspan="5">STM32F0x1<br/>STM32F0x2<br/>STM32F0x8</td>
<td style="text-align: center">STM32F03x</td>
<td style="text-align: center" rowspan="5">0x4001 5800</td>
<td style="text-align: center">0x444</td>
<td style="text-align: center" rowspan="5">0x1FFF F7CC</td>
<td style="text-align: center" rowspan="5">0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center">STM32F04x</td>
<td style="text-align: center">0x445</td>
</tr>
<tr>
<td style="text-align: center">STM32F05x</td>
<td style="text-align: center">0x440</td>
</tr>
<tr>
<td style="text-align: center">STM32F07x</td>
<td style="text-align: center">0x448</td>
</tr>
<tr>
<td style="text-align: center">STM32F09x</td>
<td style="text-align: center">0x442</td>
</tr>
<tr>
<td style="text-align: center" rowspan="6">STM32F1</td>
<td style="text-align: center" rowspan="2">STM32F100xx</td>
<td style="text-align: center">STM32F100x4<br/>STM32F100x6<br/>STM32F100x8<br/>STM32F100xB</td>
<td style="text-align: center" rowspan="2">0xE004 2000</td>
<td style="text-align: center">0x420</td>
<td style="text-align: center" rowspan="2">0x1FFF F7E0</td>
<td style="text-align: center" rowspan="2">0x1FFF F7E8</td>
</tr>
<tr>
<td style="text-align: center">STM32F100xC<br/>STM32F100xD<br/>STM32F100xE</td>
<td style="text-align: center">0x428</td>
</tr>
<tr>
<td style="text-align: center" rowspan="4">STM32F10xxx</td>
<td style="text-align: center">STM32F101x4<br/>STM32F101x6<br/>STM32F102x4<br/>STM32F102x6<br/>STM32F103x4<br/>STM32F103x6</td>
<td style="text-align: center" rowspan="4">0xE004 2000</td>
<td style="text-align: center">0x412</td>
<td style="text-align: center" rowspan="4">0x1FFF F7E0</td>
<td style="text-align: center" rowspan="4">0x1FFF F7E8</td>
</tr>
<tr>
<td style="text-align: center">STM32F101x8<br/>STM32F101xB<br/>STM32F102x8<br/>STM32F102xB<br/>STM32F103x8<br/>STM32F103xB</td>
<td style="text-align: center">0x410</td>
</tr>
<tr>
<td style="text-align: center">STM32F101xC<br/>STM32F101xD<br/>STM32F101xE<br/>STM32F103xC<br/>STM32F103xD<br/>STM32F103xE</td>
<td style="text-align: center">0x414</td>
</tr>
<tr>
<td style="text-align: center">STM32F105xx<br/>STM32F107xx</td>
<td style="text-align: center">0x418</td>
</tr>
<tr>
<td style="text-align: center" rowspan="9">STM32F3</td>
<td style="text-align: center" >STM32F301<br/>STM32F318</td>
<td style="text-align: center">STM32F301x6<br/>STM32F301x8<br/>STM32F318x8</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x439</td>
<td style="text-align: center" >0x1FFF F7CC</td>
<td style="text-align: center" >0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center" rowspan="3">STM32F302</td>
<td style="text-align: center">STM32F302x6<br/>STM32F302x8</td>
<td style="text-align: center" rowspan="3">0xE004 2000</td>
<td style="text-align: center">0x439</td>
<td style="text-align: center" rowspan="3">0x1FFF F7CC</td>
<td style="text-align: center" rowspan="3">0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center">STM32F302xB<br/>STM32F302xC</td>
<td style="text-align: center">0x422</td>
</tr>
<tr>
<td style="text-align: center">STM32F302xD<br/>STM32F302xE</td>
<td style="text-align: center">0x446</td>
</tr>
<tr>
<td style="text-align: center" rowspan="3">STM32F303<br/>STM32F328<br/>STM32F358<br/>STM32F398</td>
<td style="text-align: center">STM32F303xB<br/>STM32F303xC<br/>STM32F358xC</td>
<td style="text-align: center" rowspan="3">0xE004 2000</td>
<td style="text-align: center">0x422</td>
<td style="text-align: center" rowspan="3">0x1FFF F7CC</td>
<td style="text-align: center" rowspan="3">0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center">STM32F303x6<br/>STM32F303x8<br/>STM32F328x8</td>
<td style="text-align: center">0x438</td>
</tr>
<tr>
<td style="text-align: center">STM32F303xD<br/>STM32F303xE<br/>STM32F398xE</td>
<td style="text-align: center">0x446</td>
</tr>
<tr>
<td style="text-align: center" >STM32F334</td>
<td style="text-align: center">STM32F334</td>
<td style="text-align: center">0xE004 2000</td>
<td style="text-align: center">0x438</td>
<td style="text-align: center" >0x1FFF F7CC</td>
<td style="text-align: center" >0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center" >STM32F37xxx</td>
<td style="text-align: center" >STM32F37xxx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center" >0x432</td>
<td style="text-align: center" >0x1FFF F7CC</td>
<td style="text-align: center" >0x1FFF F7AC</td>
</tr>
<tr>
<td style="text-align: center" >STM32F2</td>
<td style="text-align: center" >STM32F2</td>
<td style="text-align: center">STM32F2</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x411</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" rowspan="10">STM32F4</td>
<td style="text-align: center" rowspan="2">STM32F401</td>
<td style="text-align: center">STM32F401xB<br/>STM32F401xC</td>
<td style="text-align: center" rowspan="2">0xE004 2000</td>
<td style="text-align: center">0x423</td>
<td style="text-align: center" rowspan="2">0x1FFF 7A22</td>
<td style="text-align: center" rowspan="2">0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center">STM32F401xD<br/>STM32F401xE</td>
<td style="text-align: center">0x433</td>
</tr>
<tr>
<td style="text-align: center" >STM32F410</td>
<td style="text-align: center">STM32F410</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x458</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" >STM32F411</td>
<td style="text-align: center">STM32F411</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x431</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" >STM32F412</td>
<td style="text-align: center">STM32F412</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x441</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" >STM32F413<br/>STM32F423</td>
<td style="text-align: center">STM32F413<br/>STM32F423</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x463</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32F405<br/>STM32F415<br/>STM32F407<br/>STM32F417<br/>STM32F427<br/>STM32F437<br/>STM32F429<br/>STM32F439</td>
<td style="text-align: center">STM32F405xx<br/>STM32F407xx<br/>STM32F415xx<br/>STM32F417xx</td>
<td style="text-align: center" rowspan="2">0xE004 2000</td>
<td style="text-align: center">0x413</td>
<td style="text-align: center" rowspan="2">0x1FFF 7A22</td>
<td style="text-align: center" rowspan="2">0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center">STM32F42xxx<br/>STM32F43xxx</td>
<td style="text-align: center">0x419</td>
</tr>
<tr>
<td style="text-align: center" >STM32F446</td>
<td style="text-align: center">STM32F446</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x421</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" >STM32F469<br/>STM32F479</td>
<td style="text-align: center">STM32F469<br/>STM32F479</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x434</td>
<td style="text-align: center" >0x1FFF 7A22</td>
<td style="text-align: center" >0x1FFF 7A10</td>
</tr>
<tr>
<td style="text-align: center" rowspan="3">STM32F7</td>
<td style="text-align: center" >STM32F72<br/>STM32F73</td>
<td style="text-align: center">STM32F72xxx<br/>STM32F73xxx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x452</td>
<td style="text-align: center" >0x1FF0 7A22</td>
<td style="text-align: center" >0x1FF0 7A10</td>
</tr>
<tr>
<td style="text-align: center" >STM32F74<br/>STM32F75</td>
<td style="text-align: center">STM32F74xxx<br/>STM32F75xxx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x449</td>
<td style="text-align: center" >0x1FF0 F442</td>
<td style="text-align: center" >0x1FF0 F420</td>
</tr>
<tr>
<td style="text-align: center" >STM32F76<br/>STM32F77</td>
<td style="text-align: center">STM32F76xxx<br/>STM32F77xxx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x451</td>
<td style="text-align: center" >0x1FF0 F442</td>
<td style="text-align: center" >0x1FF0 F420</td>
</tr>
<tr>
<td style="text-align: center" rowspan="4">STM32H7</td>
<td style="text-align: center" >STM32H72<br/>STM32H73</td>
<td style="text-align: center">STM32H72x<br/>STM32H73x</td>
<td style="text-align: center" >0xE00E 1000<br/>0x5C00 1000</td>
<td style="text-align: center">0x483</td>
<td style="text-align: center" >0x1FF1 E880</td>
<td style="text-align: center" >0x1FF1 E800</td>
</tr>
<tr>
<td style="text-align: center" >STM32H742<br/>STM32H743<br/>STM32H753<br/>STM32H750</td>
<td style="text-align: center">STM32H742<br/>STM32H743<br/>STM32H753<br/>STM32H750</td>
<td style="text-align: center" >0xE00E 1000<br/>0x5C00 1000</td>
<td style="text-align: center">0x450</td>
<td style="text-align: center" >0x1FF1 E880</td>
<td style="text-align: center" >0x1FF1 E800</td>
</tr>
<tr>
<td style="text-align: center" >STM32H745<br/>STM32H755<br/>STM32H747<br/>STM32H757</td>
<td style="text-align: center">STM32H745<br/>STM32H755<br/>STM32H747<br/>STM32H757</td>
<td style="text-align: center" >0xE00E 1000<br/>0x5C00 1000</td>
<td style="text-align: center">0x450</td>
<td style="text-align: center" >0x1FF1 E880</td>
<td style="text-align: center" >0x1FF1 E800</td>
</tr>
<tr>
<td style="text-align: center" >STM32H7A3<br/>STM32H7B3<br/>STM32H7B0</td>
<td style="text-align: center">STM32H7A3<br/>STM32H7B3<br/>STM32H7B0</td>
<td style="text-align: center" >0xE00E 1000<br/>0x5C00 1000</td>
<td style="text-align: center">0x480</td>
<td style="text-align: center" >0x08FF F80C</td>
<td style="text-align: center" >0x08FF F800</td>
</tr>
<tr>
<td style="text-align: center" rowspan="12">STM32L0</td>
<td style="text-align: center" rowspan="4">STM32L0x0</td>
<td style="text-align: center">STM32L010x3<br/>STM32L010x4</td>
<td style="text-align: center" rowspan="4">0x4001 5800</td>
<td style="text-align: center">0x457</td>
<td style="text-align: center" rowspan="4">0x1FF8 007C</td>
<td style="text-align: center" rowspan="4">0x1FF8 0050</td>
</tr>
<tr>
<td style="text-align: center">STM32L010x6</td>
<td style="text-align: center">0x425</td>
</tr>
<tr>
<td style="text-align: center">STM32L010x8</td>
<td style="text-align: center">0x417</td>
</tr>
<tr>
<td style="text-align: center">STM32L010xB</td>
<td style="text-align: center">0x447</td>
</tr>
<tr>
<td style="text-align: center" rowspan="4">STM32L0x1</td>
<td style="text-align: center">STM32L011x<br/>STM32L021x</td>
<td style="text-align: center" rowspan="4">0x4001 5800</td>
<td style="text-align: center">0x457</td>
<td style="text-align: center" rowspan="4">0x1FF8 007C</td>
<td style="text-align: center" rowspan="4">0x1FF8 0050</td>
</tr>
<tr>
<td style="text-align: center">STM32L031x<br/>STM32L041x</td>
<td style="text-align: center">0x425</td>
</tr>
<tr>
<td style="text-align: center">STM32L051x</td>
<td style="text-align: center">0x417</td>
</tr>
<tr>
<td style="text-align: center">STM32L071x<br/>STM32L081x</td>
<td style="text-align: center">0x447</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32L0x2</td>
<td style="text-align: center">STM32L052x<br/>STM32L062x</td>
<td style="text-align: center" rowspan="2">0x4001 5800</td>
<td style="text-align: center">0x417</td>
<td style="text-align: center" rowspan="2">0x1FF8 007C</td>
<td style="text-align: center" rowspan="2">0x1FF8 0050</td>
</tr>
<tr>
<td style="text-align: center">STM32L072x<br/>STM32L082x</td>
<td style="text-align: center">0x447</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32L0x3</td>
<td style="text-align: center">STM32L053x<br/>STM32L063x</td>
<td style="text-align: center" rowspan="2">0x4001 5800</td>
<td style="text-align: center">0x417</td>
<td style="text-align: center" rowspan="2">0x1FF8 007C</td>
<td style="text-align: center" rowspan="2">0x1FF8 0050</td>
</tr>
<tr>
<td style="text-align: center">STM32L073x<br/>STM32L083x</td>
<td style="text-align: center">0x447</td>
</tr>
<tr>
<td style="text-align: center" rowspan="5">STM32L1</td>
<td style="text-align: center" rowspan="5">STM32L1</td>
<td style="text-align: center">STM32L100C6<br/>STM32L100R8<br/>STM32L100RB<br/>STM32L15xx6<br/>STM32L15xx8<br/>STM32L15xxB</td>
<td style="text-align: center" rowspan="5">0xE004 2000</td>
<td style="text-align: center">0x416</td>
<td style="text-align: center" rowspan="2">0x1FF8 004C</td>
<td style="text-align: center" rowspan="2">0x1FF8 0050</td>
</tr>
<tr>
<td style="text-align: center">STM32L100C6-A<br/>STM32L100R8-A<br/>STM32L100RB-A<br/>STM32L15xx6-A<br/>STM32L15xx8-A<br/>STM32L15xxB-A</td>
<td style="text-align: center">0x429</td>
</tr>
<tr>
<td style="text-align: center">STM32L100RC<br/>STM32L15xCC<br/>STM32L15xUC<br/>STM32L15xRC<br/>STM32L15xVC<br/>STM32L162RC<br/>STM32L162VC</td>
<td style="text-align: center">0x427</td>
<td style="text-align: center" rowspan="3">0x1FF8 00CC</td>
<td style="text-align: center" rowspan="3">0x1FF8 00D0</td>
</tr>
<tr>
<td style="text-align: center">STM32L15xRCY<br/>STM32L15xRC-A<br/>STM32L15xVC-A<br/>STM32L15xQC<br/>STM32L15xZC<br/>STM32L15xRD<br/>STM32L15xVD<br/>STM32L15xQD<br/>STM32L15xZD<br/>STM32L162RC-A<br/>STM32L162VC-A<br/>STM32L162QC<br/>STM32L162ZC<br/>STM32L162RD<br/>STM32L162VD<br/>STM32L162QD<br/>STM32L162ZD</td>
<td style="text-align: center">0x436</td>
</tr>
<tr>
<td style="text-align: center">STM32L15xxE<br/>STM32L15xVD-X<br/>STM32L162xE<br/>STM32L162VD-X</td>
<td style="text-align: center">0x437</td>
</tr>
<tr>
<td style="text-align: center" rowspan="7">STM32L4</td>
<td style="text-align: center" rowspan="3">STM32L41<br/>STM32L42<br/>STM32L43<br/>STM32L44<br/>STM32L45<br/>STM32L46</td>
<td style="text-align: center">STM32L43xxx<br/>STM32L44xxx</td>
<td style="text-align: center" rowspan="3">0xE004 2000</td>
<td style="text-align: center">0x435</td>
<td style="text-align: center" rowspan="3">0x1FFF 75E0</td>
<td style="text-align: center" rowspan="3">0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center">STM32L45xxx<br/>STM32L46xxx</td>
<td style="text-align: center">0x462</td>
</tr>
<tr>
<td style="text-align: center">STM32L41xxx<br/>STM32L42xxx</td>
<td style="text-align: center">0x464</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32L47<br/>STM32L48<br/>STM32L49<br/>STM32L4A</td>
<td style="text-align: center">STM32L49xxx<br/>STM32L4Axxx</td>
<td style="text-align: center" rowspan="2">0xE004 2000</td>
<td style="text-align: center">0x461</td>
<td style="text-align: center" rowspan="2">0x1FFF 75E0</td>
<td style="text-align: center" rowspan="2">0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center">STM32L47xxx<br/>STM32L48xxx</td>
<td style="text-align: center">0x415</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32L4+</td>
<td style="text-align: center">STM32L4Rxxx<br/>STM32L4Sxxx</td>
<td style="text-align: center" rowspan="2">0xE004 2000</td>
<td style="text-align: center">0x470</td>
<td style="text-align: center" rowspan="2">0x1FFF 75E0</td>
<td style="text-align: center" rowspan="2">0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center">STM32L4P5xx<br/>STM32L4Q5xx</td>
<td style="text-align: center">0x471</td>
</tr>
<tr>
<td style="text-align: center" >STM32L5</td>
<td style="text-align: center" >STM32L5X2</td>
<td style="text-align: center">STM32L552xx<br/>STM32L562xx</td>
<td style="text-align: center" >0xE004 4000</td>
<td style="text-align: center">0x472</td>
<td style="text-align: center" >0x0BFA 05E0</td>
<td style="text-align: center" >0x0BFA 0590</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">STM32WB</td>
<td style="text-align: center" >STM32WBx0</td>
<td style="text-align: center">STM32WB50CG<br/>STM32WB30CE</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x495</td>
<td style="text-align: center" >0x1FFF 75E0</td>
<td style="text-align: center" >0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center" >STM32WB55</td>
<td style="text-align: center">STM32WB55xx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x495</td>
<td style="text-align: center" >0x1FFF 75E0</td>
<td style="text-align: center" >0x1FFF 7590</td>
</tr>
<tr>
<td style="text-align: center" >STM32WLE</td>
<td style="text-align: center" >STM32WLE</td>
<td style="text-align: center">STM32WLEx</td>
<td style="text-align: center" >0xE004 2000</td>
<td style="text-align: center">0x497</td>
<td style="text-align: center" >0x1FFF 75E0</td>
<td style="text-align: center" >0x1FFF 7590</td>
</tr>
</tbody>
</table>
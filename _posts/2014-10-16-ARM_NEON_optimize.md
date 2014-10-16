---
layout: post
title: ARM neon optimize
categories: [Development]
---

{{ page.title }}
================
最近一段时间在做性能优化的工作，考虑到算法和代码的优化已经比较艰难，准备用一些比较极端的手段来提升一下性能。
因为对汇编也没有足够多的学习和研究，所以准备工作上略显不足，也不能好好考虑到各条指令对cpu的流水线的影响。优化的工作上进展不是很快，效果也不是很好。
为了减少不必要的一些编译上的问题，以及汇编函数跳转上的问题，直接采用了内联汇编。把最耗时的计算过程用更加高效的neon的指令来执行。
　　下面先秀一段 alpha融合的指令：
　　

    asm volatile (
					"VPUSH {Q0, Q1, Q2}\n"

					"ADD R4, %[srca], %[kOffSet]\n"
					"ADD R5, %[srcc], %[iOffSet]\n"
					//Q0 source color
					"VLD1.8 {D0[0]}, [R4]\n"
					"VDUP.16 D4, D0[0]\n"		//16bit alpha in 64bit Q2[D4 D5]
					"VSHL.I32 D0, D0, #24\n"
					"VLD1.32 {D1[0]}, [R5]\n"
					//Q1 -> D3 SRC ARGB
					"VMOV.I32 D3, #0x00FFFFFF\n"
					"VAND D3, D3, D1\n"
					"VORR D3, D3, D0\n"

					"ADD R4, %[taga], %[kOffSet]\n"
					"ADD R5, %[tagc], %[iOffSet]\n"
					//Q0 target color
					"VLD1.8 {D0[0]}, [R4]\n"
					"VSHL.I32 D0, D0, #24\n"
					"VLD1.32 {D1[0]}, [R5]\n"
					//Q1 -> D2 DST ABGR
					"VMOV.I32 D2, #0x00FFFFFF\n"
					"VAND D2, D2, D1\n"
					"VORR D2, D2, D0\n"

					//Q1 Q2[D4] USED
					//BLEND START
					"VMOVL.U8 Q0, D2\n"		//8byte -> 1616byte
					"VMOVL.U8 Q1, D3\n"		//D0 D2
					"VMOV D1, D0\n"
					"VMOV D3, D2\n"
					"VQSUB.U8 D1, D2\n"		//Saturated subtract
					"VQSUB.U8 D3, D0\n"		//D1 D3

					"VMUL.I16 D1, D1, D4\n"	//Alpha * (Overlay-Source)
					"VMUL.I16 D3, D3, D4\n"	//Alpha * (Source-Overlay)

					"VSHR.S16 D1, D1, #8\n"	// >> 8
					"VSHR.S16 D3, D3, #8\n"

					"VQADD.U8 D0, D0, D3\n"	//saturated additive (Source-Overlay) < 0
					"VQSUB.U8 D0, D0, D1\n"	//saturated additive (Overlay-Source) > 0

					"VMOVN.I16 D0, Q0\n"	//16bit -> 8bit

					"VST1.16 {D0[0]}, [R5]\n"
					"ADD R5, #2\n"
					"VST1.8 {D0[2]}, [R5]\n"
					"VST1.8 {D0[3]}, [R4]\n"
					//end
					"VPOP {Q0, Q1, Q2}\n"
					:
					:[taga]"r"(pDataTagAlpha),[tagc]"r"(pDataTag),[srca]"r"(pDataSrcAlpha),[srcc]"r"(pDataSrc),[iOffSet]"r"(i),[kOffSet]"r"(k)
					:"r4", "r5"
				);

上面的代码相当于把1个RGBA的像素进行的alpha融合。pDataTagAlpha是目标图的Alpha通道，pDataTag是目标图的RGB888通道，类似的Src是源。因为这边的数据存储上，alpha和RGB是分离的，所以融合之前多了一些RGB和alpha重新交织的代码。  
内联汇编的基础知识网上有一些，但是详解没有找到，gcc官网有详细的指导，但是也是穿插在gcc编译器选项，以及汇编中，总感觉看得云里雾里。  
我碰到的要注意的地方就两个：
1. 汇编中用过的通用寄存器要记得在最后声明改变。
2. 输入列表中的指示为“r”的寄存器，不能在汇编代码里被更改。

上面的汇编 对neon寄存器来说只用到一点点，根本没有发挥出应有的能力，改造一下可以一次处理4个RGBA的融合。让性能瞬间提升。
当然上面的代码也仅仅只是执行通过，如有更好的指令能优化和使用，敬请邮件告知。
　　



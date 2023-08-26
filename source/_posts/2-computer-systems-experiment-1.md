---
url: computer-systems-experiment-1
title: 计算机系统基础实验一
copyright: true
date: 2020-02-22 10:59:33
categories: [技术]
tags: [计算机系统实验]
---
Computer Systems Experiment #1 - 编写一个C程序：filedump，用于显示规定格式的某个文件的实际字节信息。

<!--more-->

> - 实际需求如下：假定你编写的程序，编译后为filedump.exe, 需要显示内容的文件名时file1.dat，在控台上输入以下命令，注意输出信息要清晰规整。无法显示的ASCII码用？号或escape字符显示，参照ASCII码表红色标记的部分。不符合以下的输入内容，需要显示程序的使用方法。
> 1. Filedump -x file1.dat   将显示如下内容的信息，每行显示16个字节的内容，首行显示字节的ASCII码结果(小于0x20用ASCII码表的替代字符)，接下的一行为该字节的16进制数值，大于16字节时重复这种方式显示信息。
> 2. Filedump -X file1.dat   将显示如下内容的信息，每行显示16个字节的16进制内容，空2个单角空格，继续显示这16个字节的ASCII码。大于16字节时重复这种方式显示信息。不可显示字符用句点代替 “.”
> 3. Filedump -d file1.dat   将显示如下内容的信息，每行显示16个字节的内容，首行显示字节的ASCII码结果(小于0x20用ASCII码表的替代字符) ，接下的一行为该字节的10进制数值，大于16字节时重复这种方式显示信息。
> 4. Filedump -D file1.dat   将显示如下内容的信息，每行显示16个字节的10进制内容，空2个单角空格，继续显示这16个字节的ASCII码。大于16字节时重复这种方式显示信息。不可显示字符用句点代替 “.”
> 5. Filedump -b file1.dat   将显示如下内容的信息，每行显示4个字节的内容，首行显示字节的ASCII码结果(小于0x20用ASCII码表的替代字符)，接下的一行为该字节的二进制数值，大于4字节时重复这种方式显示信息。
> 6. Filedump -B file1.dat   将显示如下内容的信息，每行显示4个字节的二进制内容，空2个单角空格，继续显示这4个字节的ASCII码。大于4字节时重复这种方式显示信息。不可显示字符用句点代替 “.”

首先，题目要求编写一个C程序，为此需要掌握C程序的语法，尤其是`printf()`函数等有别于C++的地方。
[轻触以了解argc与argv的用法](https://docs.microsoft.com/en-us/cpp/c-language/arguments-to-main?view=vs-2019)
[轻触以了解printf等函数的用法](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/printf-printf-l-wprintf-wprintf-l?view=vs-2019)
[轻触以了解C字符串比较函数的用法](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/strcmp-wcscmp-mbscmp?view=vs-2019)
题目的要求就是将一个已知的文件用此程序展开，其输出为文件中字符的ASCII码结果、16进制结果、10进制结果和2进制结果等形式，并符合对输出格式的要求。
最初应该建立起程序的框架，即对应不同的选项（`-x`、`-X`、`-d`、`-D`、`-b`与`-B`），程序可以输出不同的结果，应该考虑用`if`分支或`switch`分支。
而对于数量较多的情况，最好使用`switch`分支。但在判断输入情况时，若使用`strcmp`函数比较，则使用`if`分支更好，所以这里选择用`if`。
要注意，为了避免输入其他格式造成不可预知的后果，这里应该考虑各种可能输入的情况，并对于不符合要求的情况，输出正确的形式以引导。
接下来就是对文件的读取。
[轻触以了解fopen等函数的用法](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/fopen-wfopen?view=vs-2019)
每读取一个字符，应该根据输入的选项，输出相应的格式。由于输出为整行输出，应该先把字符存入数组，接下来则对数组进行打印。为适应多种情况输出，应该构造不同函数来实现。
同样，对于文件中的特殊字符，应该考虑到其输出格式，而不应该输出乱码。读取文件时，应该注意`argv[]`角标是2而不是1。
为了使输出格式尽量整齐美观，对于打印字符的函数，分为了三种情况，分别对应输出双行、双列以及2进制数的情况。
对于小于0x20的情况，应把替代字符预先存入数组以便于输出。若输出只有一行或者两行的情况，应该考虑数组是否全面。
另外要注意`char`型变量的范围，必要时使用`unsigned char`来声明变量，防止判断条件时出错。
老师的代码中用下面的方式输出2进制；

    printf("%c", (line[i] & (0x80 >> j)) != 0 ? '1' : '0');
和移位后0x80（10000000右移j位）取与位运算，比对2进制位1和0来将其转换为2进制。
用`printf()`函数打印`char`型变量时，如果最高位是1，即超过0x7f，当%x格式化输出时会将其扩展到`int`型的32位。这也是输出出现fffffff的原因。
写完程序后可以用`file.dat`文件测试程序的功能是否完善，输出存放在`list-x.txt`文件中。
测试时应注意，exe可执行文件应与测试文件（如`file.dat`）位于同一文件夹下，否则需要输入文件相对路径。若使用VSCode，应保证.vscode文件夹下只有json文件，而其他文件位于其之外。

最后，我的解决方案如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void printASCIICharleft(char*, int);
void printASCIICharright(char*, int);
void printASCIICharbin(char*, int);
void printDec(char*, int, char*);
void printBin(char*, int);
const int LINELEN1 = 16;
const int LINELEN2 = 4;
int len = 16;
char *ascii[]={ \
    "NUL", "SOH", "STX", "ETX", "EOT",
    "ENQ", "ACK", "BEL", "BS", "TAB",
    "LF", "VT", "FF", "CR", "SO",
    "SI", "DLE", "DC1", "DC2", "DC3",
    "DC4", "NAK", "SYN", "ETB", "CAN",
    "EM", "SUB", "ESC", "FS", "GS",
    "RS", "US", " ", "!"};

int main(int argc, char *argv[])
{
    unsigned char data;
    int loc = 0;
    char line1[LINELEN1];
    char line2[LINELEN2];
    // receive arguments from command line
    if (argc != 3) {
        // the usage was wrong
        printf("ERROR! Usage: \"filedump -xXdDbB filename\"\n");
        printf("-x/X to dump by hexadecimal\n");
        printf("-d/D to dump by decimal\n");
        printf("-b/B to dump by binary\n");
        printf("filename is name of the file whose contents you want to dump\n");
        return -1;
    } else if ( strcmp(argv[1], "-x") &&
                strcmp(argv[1], "-X") &&
                strcmp(argv[1], "-d") &&
                strcmp(argv[1], "-D") &&
                strcmp(argv[1], "-b") &&
                strcmp(argv[1], "-B") ) {
                    // wrong usage too
                    printf("ERROR! Usage: \"filedump -xXdDbB filename\"\n");
                    printf("-x/X to dump by hexadecimal\n");
                    printf("-d/D to dump by decimal\n");
                    printf("-b/B to dump by binary\n");
                    return -2;
    }

    // creat datafile from argv[2]

    // open file for binary read
    FILE *fp = NULL;
    // read binary mode
    fp = fopen(argv[2], "rb");
    if (fp == NULL) {
        // failed opening
        printf("failed when opening %s\n", argv[2]);
        return -3;
    }

    while (!feof(fp)) {
        // get 1 byte of char
        data = fgetc(fp);

        // define linelength
        if (!strcmp(argv[1], "-x") ||
            !strcmp(argv[1], "-X") ||
            !strcmp(argv[1], "-d") ||
            !strcmp(argv[1], "-D")) {
            len = LINELEN1;

            if (loc < len && (!feof(fp))) {
                line1[loc] = data;
                loc++;
            } else {
                // print linelen chars
                if (!strcmp(argv[1], "-x")) {
                    printASCIICharleft(line1, len);
                    printDec(line1, len, "%2.2x\t");
                    printf("\n");
                } else if (!strcmp(argv[1], "-X")) {
                    printDec(line1, len, "%2.2x\t");
                    printf("  ");
                    printASCIICharright(line1, len);
                } else if (!strcmp(argv[1], "-d")) {
                    printASCIICharleft(line1, len);
                    printDec(line1, len, "%03d\t");
                    printf("\n");
                } else if (!strcmp(argv[1], "-D")) {
                    printDec(line1, len, "%03d\t");
                    printf("  ");
                    printASCIICharright(line1, len);
                }

                loc = 0;
                line1[loc] = data;
                loc++;
            }
        } else {
            len = LINELEN2;

            if (loc < len && (!feof(fp))) {
                line2[loc] = data;
                loc++;
            } else {
                // print linelen chars
                if (!strcmp(argv[1], "-b")) {
                    printASCIICharbin(line2, len);
                    printBin(line2, len);
                    printf("\n");
                } else if (!strcmp(argv[1], "-B")) {
                    printBin(line2, len);
                    printf("  ");
                    printASCIICharright(line2, len);
                }

                loc = 0;
                line2[loc] = data;
                loc++;
            }
        }
    }

    fclose(fp);
    return 0;
}

// print ASCII char
void printASCIICharleft(char *line, int linelen)
{
    char eachData;
    for (int i = 0; i < linelen; i++) {
        eachData = line[i];
        if (eachData < 0)
            printf("%c\t", '.');
        else if (eachData <= 0x20)
            printf("%s\t", ascii[(unsigned char)eachData]); // unsigned char
        else
            printf("%c\t", eachData);
    }
    printf("\n");
}

void printASCIICharright(char *line, int linelen)
{
    char eachData;
    for (int i = 0; i < linelen; i++) {
        eachData = line[i];
        if (eachData < 0)
            printf("%c", '.');
        else if (eachData <= 0x20)
            printf("%s", ascii[(unsigned char)eachData]); // unsigned char
        else
            printf("%c", eachData);
    }
    printf("\n");
}

void printASCIICharbin(char *line, int linelen)
{
    char eachData;
    for (int i = 0; i < linelen; i++) {
        eachData = line[i];
        if (eachData < 0)
            printf("%c\t\t", '.');
        else if (eachData <= 0x20)
            printf("%s\t\t", ascii[(unsigned char)eachData]); // unsigned char
        else
            printf("%c\t\t", eachData);
    }
    printf("\n");
}

// print decimal && hexadecimal
void printDec(char *line, int linelen, char *fmt)
{
    for (int i = 0; i < linelen; i++)
        printf(fmt, (unsigned char)line[i]); // unsigned char
}

// print binary
void printBin(char *line, int linelen)
{
    for (int i = 0; i < linelen; i++) {
        for (int j = 0; j < 8; j++)
            printf("%c", (line[i] & (0x80 >> j)) != 0 ? '1' : '0'); // get binary method
        printf("\t");
    }
}
```

对应的`list-x.txt`文件如下：

```
NUL	SOH	STX	ETX	EOT	ENQ	ACK	BEL	BS	TAB	LF	VT	FF	CR	SO	SI
00	01	02	03	04	05	06	07	08	09	0a	0b	0c	0d	0e	0f
DLE	DC1	DC2	DC3	DC4	NAK	SYN	ETB	CAN	EM	SUB	ESC	FS	GS	RS	US
10	11	12	13	14	15	16	17	18	19	1a	1b	1c	1d	1e	1f
 	!	"	#	$	%	&	'	(	)	*	+	,	-	.	/
20	21	22	23	24	25	26	27	28	29	2a	2b	2c	2d	2e	2f
0	1	2	3	4	5	6	7	8	9	:	;	<	=	>	?
30	31	32	33	34	35	36	37	38	39	3a	3b	3c	3d	3e	3f
@	A	B	C	D	E	F	G	H	I	J	K	L	M	N	O
40	41	42	43	44	45	46	47	48	49	4a	4b	4c	4d	4e	4f
P	Q	R	S	T	U	V	W	X	Y	Z	[	\	]	^	_
50	51	52	53	54	55	56	57	58	59	5a	5b	5c	5d	5e	5f
`	a	b	c	d	e	f	g	h	i	j	k	l	m	n	o
60	61	62	63	64	65	66	67	68	69	6a	6b	6c	6d	6e	6f
p	q	r	s	t	u	v	w	x	y	z	{	|	}	~	
70	71	72	73	74	75	76	77	78	79	7a	7b	7c	7d	7e	7f
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
80	81	82	83	84	85	86	87	88	89	8a	8b	8c	8d	8e	8f
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
90	91	92	93	94	95	96	97	98	99	9a	9b	9c	9d	9e	9f
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
a0	a1	a2	a3	a4	a5	a6	a7	a8	a9	aa	ab	ac	ad	ae	af
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
b0	b1	b2	b3	b4	b5	b6	b7	b8	b9	ba	bb	bc	bd	be	bf
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
c0	c1	c2	c3	c4	c5	c6	c7	c8	c9	ca	cb	cc	cd	ce	cf
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
d0	d1	d2	d3	d4	d5	d6	d7	d8	d9	da	db	dc	dd	de	df
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
e0	e1	e2	e3	e4	e5	e6	e7	e8	e9	ea	eb	ec	ed	ee	ef
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
f0	f1	f2	f3	f4	f5	f6	f7	f8	f9	fa	fb	fc	fd	fe	ff
00	01	02	03	04	05	06	07	08	09	0a	0b	0c	0d	0e	0f	  NULSOHSTXETXEOTENQACKBELBSTABLFVTFFCRSOSI
10	11	12	13	14	15	16	17	18	19	1a	1b	1c	1d	1e	1f	  DLEDC1DC2DC3DC4NAKSYNETBCANEMSUBESCFSGSRSUS
20	21	22	23	24	25	26	27	28	29	2a	2b	2c	2d	2e	2f	   !"#$%&'()*+,-./
30	31	32	33	34	35	36	37	38	39	3a	3b	3c	3d	3e	3f	  0123456789:;<=>?
40	41	42	43	44	45	46	47	48	49	4a	4b	4c	4d	4e	4f	  @ABCDEFGHIJKLMNO
50	51	52	53	54	55	56	57	58	59	5a	5b	5c	5d	5e	5f	  PQRSTUVWXYZ[\]^_
60	61	62	63	64	65	66	67	68	69	6a	6b	6c	6d	6e	6f	  `abcdefghijklmno
70	71	72	73	74	75	76	77	78	79	7a	7b	7c	7d	7e	7f	  pqrstuvwxyz{|}~
80	81	82	83	84	85	86	87	88	89	8a	8b	8c	8d	8e	8f	  ................
90	91	92	93	94	95	96	97	98	99	9a	9b	9c	9d	9e	9f	  ................
a0	a1	a2	a3	a4	a5	a6	a7	a8	a9	aa	ab	ac	ad	ae	af	  ................
b0	b1	b2	b3	b4	b5	b6	b7	b8	b9	ba	bb	bc	bd	be	bf	  ................
c0	c1	c2	c3	c4	c5	c6	c7	c8	c9	ca	cb	cc	cd	ce	cf	  ................
d0	d1	d2	d3	d4	d5	d6	d7	d8	d9	da	db	dc	dd	de	df	  ................
e0	e1	e2	e3	e4	e5	e6	e7	e8	e9	ea	eb	ec	ed	ee	ef	  ................
f0	f1	f2	f3	f4	f5	f6	f7	f8	f9	fa	fb	fc	fd	fe	ff	  ................
NUL	SOH	STX	ETX	EOT	ENQ	ACK	BEL	BS	TAB	LF	VT	FF	CR	SO	SI
000	001	002	003	004	005	006	007	008	009	010	011	012	013	014	015
DLE	DC1	DC2	DC3	DC4	NAK	SYN	ETB	CAN	EM	SUB	ESC	FS	GS	RS	US
016	017	018	019	020	021	022	023	024	025	026	027	028	029	030	031
 	!	"	#	$	%	&	'	(	)	*	+	,	-	.	/
032	033	034	035	036	037	038	039	040	041	042	043	044	045	046	047
0	1	2	3	4	5	6	7	8	9	:	;	<	=	>	?
048	049	050	051	052	053	054	055	056	057	058	059	060	061	062	063
@	A	B	C	D	E	F	G	H	I	J	K	L	M	N	O
064	065	066	067	068	069	070	071	072	073	074	075	076	077	078	079
P	Q	R	S	T	U	V	W	X	Y	Z	[	\	]	^	_
080	081	082	083	084	085	086	087	088	089	090	091	092	093	094	095
`	a	b	c	d	e	f	g	h	i	j	k	l	m	n	o
096	097	098	099	100	101	102	103	104	105	106	107	108	109	110	111
p	q	r	s	t	u	v	w	x	y	z	{	|	}	~	
112	113	114	115	116	117	118	119	120	121	122	123	124	125	126	127
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
128	129	130	131	132	133	134	135	136	137	138	139	140	141	142	143
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
144	145	146	147	148	149	150	151	152	153	154	155	156	157	158	159
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
160	161	162	163	164	165	166	167	168	169	170	171	172	173	174	175
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
176	177	178	179	180	181	182	183	184	185	186	187	188	189	190	191
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
192	193	194	195	196	197	198	199	200	201	202	203	204	205	206	207
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
208	209	210	211	212	213	214	215	216	217	218	219	220	221	222	223
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
224	225	226	227	228	229	230	231	232	233	234	235	236	237	238	239
.	.	.	.	.	.	.	.	.	.	.	.	.	.	.	.
240	241	242	243	244	245	246	247	248	249	250	251	252	253	254	255
000	001	002	003	004	005	006	007	008	009	010	011	012	013	014	015	  NULSOHSTXETXEOTENQACKBELBSTABLFVTFFCRSOSI
016	017	018	019	020	021	022	023	024	025	026	027	028	029	030	031	  DLEDC1DC2DC3DC4NAKSYNETBCANEMSUBESCFSGSRSUS
032	033	034	035	036	037	038	039	040	041	042	043	044	045	046	047	   !"#$%&'()*+,-./
048	049	050	051	052	053	054	055	056	057	058	059	060	061	062	063	  0123456789:;<=>?
064	065	066	067	068	069	070	071	072	073	074	075	076	077	078	079	  @ABCDEFGHIJKLMNO
080	081	082	083	084	085	086	087	088	089	090	091	092	093	094	095	  PQRSTUVWXYZ[\]^_
096	097	098	099	100	101	102	103	104	105	106	107	108	109	110	111	  `abcdefghijklmno
112	113	114	115	116	117	118	119	120	121	122	123	124	125	126	127	  pqrstuvwxyz{|}~
128	129	130	131	132	133	134	135	136	137	138	139	140	141	142	143	  ................
144	145	146	147	148	149	150	151	152	153	154	155	156	157	158	159	  ................
160	161	162	163	164	165	166	167	168	169	170	171	172	173	174	175	  ................
176	177	178	179	180	181	182	183	184	185	186	187	188	189	190	191	  ................
192	193	194	195	196	197	198	199	200	201	202	203	204	205	206	207	  ................
208	209	210	211	212	213	214	215	216	217	218	219	220	221	222	223	  ................
224	225	226	227	228	229	230	231	232	233	234	235	236	237	238	239	  ................
240	241	242	243	244	245	246	247	248	249	250	251	252	253	254	255	  ................
NUL		SOH		STX		ETX
00000000	00000001	00000010	00000011
EOT		ENQ		ACK		BEL
00000100	00000101	00000110	00000111
BS		TAB		LF		VT
00001000	00001001	00001010	00001011
FF		CR		SO		SI
00001100	00001101	00001110	00001111
DLE		DC1		DC2		DC3
00010000	00010001	00010010	00010011
DC4		NAK		SYN		ETB
00010100	00010101	00010110	00010111
CAN		EM		SUB		ESC
00011000	00011001	00011010	00011011
FS		GS		RS		US
00011100	00011101	00011110	00011111
 		!		"		#
00100000	00100001	00100010	00100011
$		%		&		'
00100100	00100101	00100110	00100111
(		)		*		+
00101000	00101001	00101010	00101011
,		-		.		/
00101100	00101101	00101110	00101111
0		1		2		3
00110000	00110001	00110010	00110011
4		5		6		7
00110100	00110101	00110110	00110111
8		9		:		;
00111000	00111001	00111010	00111011
<		=		>		?
00111100	00111101	00111110	00111111
@		A		B		C
01000000	01000001	01000010	01000011
D		E		F		G
01000100	01000101	01000110	01000111
H		I		J		K
01001000	01001001	01001010	01001011
L		M		N		O
01001100	01001101	01001110	01001111
P		Q		R		S
01010000	01010001	01010010	01010011
T		U		V		W
01010100	01010101	01010110	01010111
X		Y		Z		[
01011000	01011001	01011010	01011011
\		]		^		_
01011100	01011101	01011110	01011111
`		a		b		c
01100000	01100001	01100010	01100011
d		e		f		g
01100100	01100101	01100110	01100111
h		i		j		k
01101000	01101001	01101010	01101011
l		m		n		o
01101100	01101101	01101110	01101111
p		q		r		s
01110000	01110001	01110010	01110011
t		u		v		w
01110100	01110101	01110110	01110111
x		y		z		{
01111000	01111001	01111010	01111011
|		}		~		
01111100	01111101	01111110	01111111
.		.		.		.
10000000	10000001	10000010	10000011
.		.		.		.
10000100	10000101	10000110	10000111
.		.		.		.
10001000	10001001	10001010	10001011
.		.		.		.
10001100	10001101	10001110	10001111
.		.		.		.
10010000	10010001	10010010	10010011
.		.		.		.
10010100	10010101	10010110	10010111
.		.		.		.
10011000	10011001	10011010	10011011
.		.		.		.
10011100	10011101	10011110	10011111
.		.		.		.
10100000	10100001	10100010	10100011
.		.		.		.
10100100	10100101	10100110	10100111
.		.		.		.
10101000	10101001	10101010	10101011
.		.		.		.
10101100	10101101	10101110	10101111
.		.		.		.
10110000	10110001	10110010	10110011
.		.		.		.
10110100	10110101	10110110	10110111
.		.		.		.
10111000	10111001	10111010	10111011
.		.		.		.
10111100	10111101	10111110	10111111
.		.		.		.
11000000	11000001	11000010	11000011
.		.		.		.
11000100	11000101	11000110	11000111
.		.		.		.
11001000	11001001	11001010	11001011
.		.		.		.
11001100	11001101	11001110	11001111
.		.		.		.
11010000	11010001	11010010	11010011
.		.		.		.
11010100	11010101	11010110	11010111
.		.		.		.
11011000	11011001	11011010	11011011
.		.		.		.
11011100	11011101	11011110	11011111
.		.		.		.
11100000	11100001	11100010	11100011
.		.		.		.
11100100	11100101	11100110	11100111
.		.		.		.
11101000	11101001	11101010	11101011
.		.		.		.
11101100	11101101	11101110	11101111
.		.		.		.
11110000	11110001	11110010	11110011
.		.		.		.
11110100	11110101	11110110	11110111
.		.		.		.
11111000	11111001	11111010	11111011
.		.		.		.
11111100	11111101	11111110	11111111
00000000	00000001	00000010	00000011	  NULSOHSTXETX
00000100	00000101	00000110	00000111	  EOTENQACKBEL
00001000	00001001	00001010	00001011	  BSTABLFVT
00001100	00001101	00001110	00001111	  FFCRSOSI
00010000	00010001	00010010	00010011	  DLEDC1DC2DC3
00010100	00010101	00010110	00010111	  DC4NAKSYNETB
00011000	00011001	00011010	00011011	  CANEMSUBESC
00011100	00011101	00011110	00011111	  FSGSRSUS
00100000	00100001	00100010	00100011	   !"#
00100100	00100101	00100110	00100111	  $%&'
00101000	00101001	00101010	00101011	  ()*+
00101100	00101101	00101110	00101111	  ,-./
00110000	00110001	00110010	00110011	  0123
00110100	00110101	00110110	00110111	  4567
00111000	00111001	00111010	00111011	  89:;
00111100	00111101	00111110	00111111	  <=>?
01000000	01000001	01000010	01000011	  @ABC
01000100	01000101	01000110	01000111	  DEFG
01001000	01001001	01001010	01001011	  HIJK
01001100	01001101	01001110	01001111	  LMNO
01010000	01010001	01010010	01010011	  PQRS
01010100	01010101	01010110	01010111	  TUVW
01011000	01011001	01011010	01011011	  XYZ[
01011100	01011101	01011110	01011111	  \]^_
01100000	01100001	01100010	01100011	  `abc
01100100	01100101	01100110	01100111	  defg
01101000	01101001	01101010	01101011	  hijk
01101100	01101101	01101110	01101111	  lmno
01110000	01110001	01110010	01110011	  pqrs
01110100	01110101	01110110	01110111	  tuvw
01111000	01111001	01111010	01111011	  xyz{
01111100	01111101	01111110	01111111	  |}~
10000000	10000001	10000010	10000011	  ....
10000100	10000101	10000110	10000111	  ....
10001000	10001001	10001010	10001011	  ....
10001100	10001101	10001110	10001111	  ....
10010000	10010001	10010010	10010011	  ....
10010100	10010101	10010110	10010111	  ....
10011000	10011001	10011010	10011011	  ....
10011100	10011101	10011110	10011111	  ....
10100000	10100001	10100010	10100011	  ....
10100100	10100101	10100110	10100111	  ....
10101000	10101001	10101010	10101011	  ....
10101100	10101101	10101110	10101111	  ....
10110000	10110001	10110010	10110011	  ....
10110100	10110101	10110110	10110111	  ....
10111000	10111001	10111010	10111011	  ....
10111100	10111101	10111110	10111111	  ....
11000000	11000001	11000010	11000011	  ....
11000100	11000101	11000110	11000111	  ....
11001000	11001001	11001010	11001011	  ....
11001100	11001101	11001110	11001111	  ....
11010000	11010001	11010010	11010011	  ....
11010100	11010101	11010110	11010111	  ....
11011000	11011001	11011010	11011011	  ....
11011100	11011101	11011110	11011111	  ....
11100000	11100001	11100010	11100011	  ....
11100100	11100101	11100110	11100111	  ....
11101000	11101001	11101010	11101011	  ....
11101100	11101101	11101110	11101111	  ....
11110000	11110001	11110010	11110011	  ....
11110100	11110101	11110110	11110111	  ....
11111000	11111001	11111010	11111011	  ....
11111100	11111101	11111110	11111111	  ....
ERROR! Usage: "filedump -xXdDbB filename"
-x/X to dump by hexadecimal
-d/D to dump by decimal
-b/B to dump by binary
ERROR! Usage: "filedump -xXdDbB filename"
-x/X to dump by hexadecimal
-d/D to dump by decimal
-b/B to dump by binary
filename is name of the file whose contents you want to dump
```

如果你有更好的解决方案，欢迎与我交流，有问题请指正。

最后的最后，附上老师的code。

```
//This code is start at 2020/2/12 during corona-virus events
//
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>

#define DF_LINEMAX (16)
#define DF_BINMODE_BYTENUM (4)
#define DF_HEXFMT "%2.2x   "
#define DF_DECFMT "%03d  "
#define DF_HEXDECSPC "%5s"
#define DF_BINSPC "%8s\t"
//define support file dump mode
typedef enum dumpxmode{
    USAGE,  //help message
    xMODE,  //up/down hex
    XMODE,  //left/right hex
    dMODE,  //up/down decimal
    DMODE,  //left/right decimal
    bMODE,  //up/down binary
    BMODE   //L/R binary
} FDUMPxmode;

// define enough buffer
char buff[DF_LINEMAX+DF_BINMODE_BYTENUM];
//prototype define
void printchar(char *buf,int maxlen,FDUMPxmode dpmode);
void printfmt(char *buf,int maxlen, char *fmt,FDUMPxmode dpmode);
void create_datfile(char * fname);
void printbin(char *buf,int maxlen,FDUMPxmode dpmode);
void processbuf(char*buf, int len, FDUMPxmode dpxmode);
int getLineLength(FDUMPxmode dpxmode);
FDUMPxmode getdpmode(char *args);
void printSpace(int cols,FDUMPxmode dpmode);

char *asciiabl[]={ \
    "NUL","SOH","STX","ETX","EOT",
    "ENQ","ACK","BEL","BS","TAB",
    "LF", "VT", "FF", "CR", "SO",
    "SI","DLE","DC1","DC2","DC3",
    "DC4","NAK","SYN","ETB","CAN",
    "EM","SUB","ESC","FS","GS","RS",
    "US","SP","!"};

int main(int argc,char * argv[])
{
    unsigned char data;
    FDUMPxmode dpxmode;
    int binlen;
    int len=0;
    int lenMAX = DF_BINMODE_BYTENUM;
     /*get arguments from command line*/
    if(argc<3)
    {
        //not input filename
        dpxmode =USAGE;
    }else{ //>=3
        dpxmode =getdpmode(argv[1]);
    }

    if(dpxmode == USAGE)
    {
        printf("Usage: filedump option file.dat\n");
        printf("option: -x|-X|-d|-D|-b|-B \n");
        printf("file.dat: the file will be dumped.\n");
        printf("\n");
        return 1;
    }

    // get output bytes number
    lenMAX= getLineLength(dpxmode);

    //create_datfile(argv[1]);

    //open file for binary read
    FILE *fp;
    //file stream open for read binary xmode
    fp=fopen(argv[2],"rb");
    if(fp==NULL)
    {
        //open failed
        printf("open file %s failed!\n",argv[1]);
        return(2);
    }

    //read file contents and printf
    len=0;
    while(!feof(fp))
    {
        //get 1byte of char
        data=fgetc(fp);

        //read DF_LINEMAX byte of chars
        if(len < lenMAX && (!feof(fp)))
        {
            buff[len]=data;
            len++;
        }else{
            //16byte of chars output
            processbuf(buff,len,dpxmode);

            len=0;
            //revise for lost 0x10 0x20 ...
            buff[len]=data;
            len++;
        }
    }


    fclose(fp);
    return 0;

}

//process read in data from FileStream
void processbuf(char*buf, int len, FDUMPxmode dpxmode)
{

    switch(dpxmode)
    {
        case xMODE: //up/down hex
            printchar(buf,len,dpxmode);
            printf("\n");
            printfmt(buf,len,"%2.2x\t",dpxmode);
            printf("\n");
            break;
        case dMODE: //up/down dec
            printchar(buf,len,dpxmode);
            printf("\n");
            printfmt(buf,len,"%03d\t",dpxmode);
            printf("\n");
            break;
        case bMODE: //up/down binary
            printchar(buf,len,dpxmode);
            printf("\n");
            printbin(buf,len,dpxmode);
            printf("\n");
            break;
        case XMODE: // /L/R hex
            printfmt(buf,len,DF_HEXFMT,dpxmode);
            printf("  ");
            printchar(buf,len,dpxmode);
            printf("\n");
        break;
        case DMODE: //L/R dec
            printfmt(buf,len,DF_DECFMT,dpxmode);
            printf("  ");
            printchar(buf,len,dpxmode);
            printf("\n");
            break;
        case BMODE: //L/R bin
            printbin(buf,len,dpxmode);
            printf("  ");
            printchar(buf,len,dpxmode);
            printf("\n");
            break;
        default:
            //never come here
            printf("Never come here!\n");
            break;
    }
    printf("\n");
}

//get output byte number
int getLineLength(FDUMPxmode dpxmode)
{
    //get read byte number
    int len;
    if(dpxmode ==bMODE || dpxmode ==BMODE)
    {
        len=DF_BINMODE_BYTENUM;
    }else{
        len= DF_LINEMAX;
    }
    return  len;
}

//get selected mode
FDUMPxmode getdpmode(char *args)
{
    FDUMPxmode dpxmode;
    if(strncmp(args,"-x",2)==0)
    {
        dpxmode =xMODE;
    }else if(strncmp(args,"-X",2)==0)
    {
        dpxmode =XMODE;
    }else if(strncmp(args,"-d",2)==0)
    {
        dpxmode =dMODE;

    }else if(strncmp(args,"-D",2)==0)
    {
        dpxmode =DMODE;

    }else if(strncmp(args,"-b",2)==0)
    {
        dpxmode =bMODE;

    }else if(strncmp(args,"-B",2)==0)
    {
        dpxmode =BMODE;

    }else{
        //usage
        dpxmode = USAGE;
    }
    return dpxmode;
}

// print char with visible ascii
void printchar(char *buf,int maxlen, FDUMPxmode dpmode)
{
    int i;
    char tempdata;

    for(i=0;i<maxlen;i++)
    {
        tempdata=buf[i];
        if(tempdata <0)
        {
            if(dpmode==XMODE || dpmode==DMODE||dpmode==BMODE)
            {
                printf("%c",'?');
            }else if(dpmode==bMODE){

                printf("%c\t\t",'?');
            }else{
                printf("%c\t",'?');

            }
        }else if(tempdata<=0x20){
            //index must unsigned
            if(dpmode==XMODE || dpmode==DMODE||dpmode==BMODE)
            {
                //L/R mode replace with DOT|?
                printf("%c",'?');
            }else if(dpmode==bMODE){
                //u/d mode replace with special named chars
                printf("%s\t\t",asciiabl[(unsigned char)tempdata]);

            }else{
                printf("%s\t",asciiabl[(unsigned char)tempdata]);
            }

        }else{
            //0x21==0x7F
            if(dpmode==XMODE || dpmode==DMODE||dpmode==BMODE)
            {
                //L/R
                //0x7F DELETE has no display font
                printf("%c",(tempdata!=0x7f)?tempdata:'?');
            }else{
                //u/d
                if(tempdata!=0x7F)
                {
                    if(dpmode==bMODE)
                    {
                        printf("%c\t\t",tempdata);
                    }
                    else
                    {
                        /* code */
                        printf("%c\t",tempdata);
                    }

                }else
                {
                    //0x7F DELETE has no display font
                    /* code */
                    if(dpmode==bMODE)
                    {
                        printf("%s\t\t","DEL");
                    }else
                    {
                        /* code */
                        printf("%s\t","DEL");
                    }

                }
            }
        }
    }


}

//print last space for L/R mode
void printSpace(int cols,FDUMPxmode dpmode)
{
    int i;
    if(dpmode ==BMODE)
    {
        for(i=0;i<cols;i++)
        {
            printf(DF_BINSPC," ");
        }

    }else{

        for(i=0;i<cols;i++)
        {
            printf(DF_HEXDECSPC," ");
        }


    }
}

// print binary of the char
// char with visible ascii
void printbin(char *buf,int maxlen,FDUMPxmode dpmode)
{
    int i,j;
    int templen=getLineLength(dpmode);
    for(i=0;i<maxlen;i++)
    {
        for(j=0;j<8;j++)
        {
            printf("%c",(buf[i]&(0x80>>j))!=0?'1':'0');
        }
        printf("\t");
    }
    //actural num < define max number

    if(maxlen<templen && maxlen!=0)
    {
        printSpace((templen-maxlen),dpmode);
    }
    // printf("\n");
}

// printf hex oct dec format
void printfmt(char *buf,int maxlen,char *fmt,FDUMPxmode dpmode)
{
    int i;
    int templen=getLineLength(dpmode);

    for(i=0;i<maxlen;i++)
    {

        printf(fmt,(unsigned char)buf[i]);

    }
    if(dpmode == DMODE || dpmode==XMODE)
    {
        if(maxlen<templen && maxlen!=0)
        {
            printSpace((templen-maxlen),dpmode);

        }
    }
    /// printf("\n");

}

//create test file with 0x00-0xff
void create_datfile(char * fname)
{
    unsigned char i=0;
    FILE *fp;
    fp=fopen(fname,"wb+");
    if(fp==NULL)
    {
        printf("File creation error!\n");
        exit(4);
    }

    for(i=0;i<UCHAR_MAX;i++)
    {
        fputc(i,fp);
    }
    fputc(i,fp);
    //fputc(i-2,fp);

    fclose(fp);
}
```

输出：

```
NUL	SOH	STX	ETX	EOT	ENQ	ACK	BEL	BS	TAB	LF	VT	FF	CR	SO	SI
00	01	02	03	04	05	06	07	08	09	0a	0b	0c	0d	0e	0f

DLE	DC1	DC2	DC3	DC4	NAK	SYN	ETB	CAN	EM	SUB	ESC	FS	GS	RS	US
10	11	12	13	14	15	16	17	18	19	1a	1b	1c	1d	1e	1f

SP	!	"	#	$	%	&	'	(	)	*	+	,	-	.	/
20	21	22	23	24	25	26	27	28	29	2a	2b	2c	2d	2e	2f

0	1	2	3	4	5	6	7	8	9	:	;	<	=	>	?
30	31	32	33	34	35	36	37	38	39	3a	3b	3c	3d	3e	3f

@	A	B	C	D	E	F	G	H	I	J	K	L	M	N	O
40	41	42	43	44	45	46	47	48	49	4a	4b	4c	4d	4e	4f

P	Q	R	S	T	U	V	W	X	Y	Z	[	\	]	^	_
50	51	52	53	54	55	56	57	58	59	5a	5b	5c	5d	5e	5f

`	a	b	c	d	e	f	g	h	i	j	k	l	m	n	o
60	61	62	63	64	65	66	67	68	69	6a	6b	6c	6d	6e	6f

p	q	r	s	t	u	v	w	x	y	z	{	|	}	~	DEL
70	71	72	73	74	75	76	77	78	79	7a	7b	7c	7d	7e	7f

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
80	81	82	83	84	85	86	87	88	89	8a	8b	8c	8d	8e	8f

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
90	91	92	93	94	95	96	97	98	99	9a	9b	9c	9d	9e	9f

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
a0	a1	a2	a3	a4	a5	a6	a7	a8	a9	aa	ab	ac	ad	ae	af

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
b0	b1	b2	b3	b4	b5	b6	b7	b8	b9	ba	bb	bc	bd	be	bf

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
c0	c1	c2	c3	c4	c5	c6	c7	c8	c9	ca	cb	cc	cd	ce	cf

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
d0	d1	d2	d3	d4	d5	d6	d7	d8	d9	da	db	dc	dd	de	df

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
e0	e1	e2	e3	e4	e5	e6	e7	e8	e9	ea	eb	ec	ed	ee	ef

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
f0	f1	f2	f3	f4	f5	f6	f7	f8	f9	fa	fb	fc	fd	fe	ff

00   01   02   03   04   05   06   07   08   09   0a   0b   0c   0d   0e   0f     ????????????????

10   11   12   13   14   15   16   17   18   19   1a   1b   1c   1d   1e   1f     ????????????????

20   21   22   23   24   25   26   27   28   29   2a   2b   2c   2d   2e   2f     ?!"#$%&'()*+,-./

30   31   32   33   34   35   36   37   38   39   3a   3b   3c   3d   3e   3f     0123456789:;<=>?

40   41   42   43   44   45   46   47   48   49   4a   4b   4c   4d   4e   4f     @ABCDEFGHIJKLMNO

50   51   52   53   54   55   56   57   58   59   5a   5b   5c   5d   5e   5f     PQRSTUVWXYZ[\]^_

60   61   62   63   64   65   66   67   68   69   6a   6b   6c   6d   6e   6f     `abcdefghijklmno

70   71   72   73   74   75   76   77   78   79   7a   7b   7c   7d   7e   7f     pqrstuvwxyz{|}~?

80   81   82   83   84   85   86   87   88   89   8a   8b   8c   8d   8e   8f     ????????????????

90   91   92   93   94   95   96   97   98   99   9a   9b   9c   9d   9e   9f     ????????????????

a0   a1   a2   a3   a4   a5   a6   a7   a8   a9   aa   ab   ac   ad   ae   af     ????????????????

b0   b1   b2   b3   b4   b5   b6   b7   b8   b9   ba   bb   bc   bd   be   bf     ????????????????

c0   c1   c2   c3   c4   c5   c6   c7   c8   c9   ca   cb   cc   cd   ce   cf     ????????????????

d0   d1   d2   d3   d4   d5   d6   d7   d8   d9   da   db   dc   dd   de   df     ????????????????

e0   e1   e2   e3   e4   e5   e6   e7   e8   e9   ea   eb   ec   ed   ee   ef     ????????????????

f0   f1   f2   f3   f4   f5   f6   f7   f8   f9   fa   fb   fc   fd   fe   ff     ????????????????

NUL	SOH	STX	ETX	EOT	ENQ	ACK	BEL	BS	TAB	LF	VT	FF	CR	SO	SI
000	001	002	003	004	005	006	007	008	009	010	011	012	013	014	015

DLE	DC1	DC2	DC3	DC4	NAK	SYN	ETB	CAN	EM	SUB	ESC	FS	GS	RS	US
016	017	018	019	020	021	022	023	024	025	026	027	028	029	030	031

SP	!	"	#	$	%	&	'	(	)	*	+	,	-	.	/
032	033	034	035	036	037	038	039	040	041	042	043	044	045	046	047

0	1	2	3	4	5	6	7	8	9	:	;	<	=	>	?
048	049	050	051	052	053	054	055	056	057	058	059	060	061	062	063

@	A	B	C	D	E	F	G	H	I	J	K	L	M	N	O
064	065	066	067	068	069	070	071	072	073	074	075	076	077	078	079

P	Q	R	S	T	U	V	W	X	Y	Z	[	\	]	^	_
080	081	082	083	084	085	086	087	088	089	090	091	092	093	094	095

`	a	b	c	d	e	f	g	h	i	j	k	l	m	n	o
096	097	098	099	100	101	102	103	104	105	106	107	108	109	110	111

p	q	r	s	t	u	v	w	x	y	z	{	|	}	~	DEL
112	113	114	115	116	117	118	119	120	121	122	123	124	125	126	127

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
128	129	130	131	132	133	134	135	136	137	138	139	140	141	142	143

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
144	145	146	147	148	149	150	151	152	153	154	155	156	157	158	159

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
160	161	162	163	164	165	166	167	168	169	170	171	172	173	174	175

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
176	177	178	179	180	181	182	183	184	185	186	187	188	189	190	191

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
192	193	194	195	196	197	198	199	200	201	202	203	204	205	206	207

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
208	209	210	211	212	213	214	215	216	217	218	219	220	221	222	223

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
224	225	226	227	228	229	230	231	232	233	234	235	236	237	238	239

?	?	?	?	?	?	?	?	?	?	?	?	?	?	?	?
240	241	242	243	244	245	246	247	248	249	250	251	252	253	254	255

000  001  002  003  004  005  006  007  008  009  010  011  012  013  014  015    ????????????????

016  017  018  019  020  021  022  023  024  025  026  027  028  029  030  031    ????????????????

032  033  034  035  036  037  038  039  040  041  042  043  044  045  046  047    ?!"#$%&'()*+,-./

048  049  050  051  052  053  054  055  056  057  058  059  060  061  062  063    0123456789:;<=>?

064  065  066  067  068  069  070  071  072  073  074  075  076  077  078  079    @ABCDEFGHIJKLMNO

080  081  082  083  084  085  086  087  088  089  090  091  092  093  094  095    PQRSTUVWXYZ[\]^_

096  097  098  099  100  101  102  103  104  105  106  107  108  109  110  111    `abcdefghijklmno

112  113  114  115  116  117  118  119  120  121  122  123  124  125  126  127    pqrstuvwxyz{|}~?

128  129  130  131  132  133  134  135  136  137  138  139  140  141  142  143    ????????????????

144  145  146  147  148  149  150  151  152  153  154  155  156  157  158  159    ????????????????

160  161  162  163  164  165  166  167  168  169  170  171  172  173  174  175    ????????????????

176  177  178  179  180  181  182  183  184  185  186  187  188  189  190  191    ????????????????

192  193  194  195  196  197  198  199  200  201  202  203  204  205  206  207    ????????????????

208  209  210  211  212  213  214  215  216  217  218  219  220  221  222  223    ????????????????

224  225  226  227  228  229  230  231  232  233  234  235  236  237  238  239    ????????????????

240  241  242  243  244  245  246  247  248  249  250  251  252  253  254  255    ????????????????

NUL		SOH		STX		ETX
00000000	00000001	00000010	00000011

EOT		ENQ		ACK		BEL
00000100	00000101	00000110	00000111

BS		TAB		LF		VT
00001000	00001001	00001010	00001011

FF		CR		SO		SI
00001100	00001101	00001110	00001111

DLE		DC1		DC2		DC3
00010000	00010001	00010010	00010011

DC4		NAK		SYN		ETB
00010100	00010101	00010110	00010111

CAN		EM		SUB		ESC
00011000	00011001	00011010	00011011

FS		GS		RS		US
00011100	00011101	00011110	00011111

SP		!		"		#
00100000	00100001	00100010	00100011

$		%		&		'
00100100	00100101	00100110	00100111

(		)		*		+
00101000	00101001	00101010	00101011

,		-		.		/
00101100	00101101	00101110	00101111

0		1		2		3
00110000	00110001	00110010	00110011

4		5		6		7
00110100	00110101	00110110	00110111

8		9		:		;
00111000	00111001	00111010	00111011

<		=		>		?
00111100	00111101	00111110	00111111

@		A		B		C
01000000	01000001	01000010	01000011

D		E		F		G
01000100	01000101	01000110	01000111

H		I		J		K
01001000	01001001	01001010	01001011

L		M		N		O
01001100	01001101	01001110	01001111

P		Q		R		S
01010000	01010001	01010010	01010011

T		U		V		W
01010100	01010101	01010110	01010111

X		Y		Z		[
01011000	01011001	01011010	01011011

\		]		^		_
01011100	01011101	01011110	01011111

`		a		b		c
01100000	01100001	01100010	01100011

d		e		f		g
01100100	01100101	01100110	01100111

h		i		j		k
01101000	01101001	01101010	01101011

l		m		n		o
01101100	01101101	01101110	01101111

p		q		r		s
01110000	01110001	01110010	01110011

t		u		v		w
01110100	01110101	01110110	01110111

x		y		z		{
01111000	01111001	01111010	01111011

|		}		~		DEL
01111100	01111101	01111110	01111111

?		?		?		?
10000000	10000001	10000010	10000011

?		?		?		?
10000100	10000101	10000110	10000111

?		?		?		?
10001000	10001001	10001010	10001011

?		?		?		?
10001100	10001101	10001110	10001111

?		?		?		?
10010000	10010001	10010010	10010011

?		?		?		?
10010100	10010101	10010110	10010111

?		?		?		?
10011000	10011001	10011010	10011011

?		?		?		?
10011100	10011101	10011110	10011111

?		?		?		?
10100000	10100001	10100010	10100011

?		?		?		?
10100100	10100101	10100110	10100111

?		?		?		?
10101000	10101001	10101010	10101011

?		?		?		?
10101100	10101101	10101110	10101111

?		?		?		?
10110000	10110001	10110010	10110011

?		?		?		?
10110100	10110101	10110110	10110111

?		?		?		?
10111000	10111001	10111010	10111011

?		?		?		?
10111100	10111101	10111110	10111111

?		?		?		?
11000000	11000001	11000010	11000011

?		?		?		?
11000100	11000101	11000110	11000111

?		?		?		?
11001000	11001001	11001010	11001011

?		?		?		?
11001100	11001101	11001110	11001111

?		?		?		?
11010000	11010001	11010010	11010011

?		?		?		?
11010100	11010101	11010110	11010111

?		?		?		?
11011000	11011001	11011010	11011011

?		?		?		?
11011100	11011101	11011110	11011111

?		?		?		?
11100000	11100001	11100010	11100011

?		?		?		?
11100100	11100101	11100110	11100111

?		?		?		?
11101000	11101001	11101010	11101011

?		?		?		?
11101100	11101101	11101110	11101111

?		?		?		?
11110000	11110001	11110010	11110011

?		?		?		?
11110100	11110101	11110110	11110111

?		?		?		?
11111000	11111001	11111010	11111011

?		?		?		?
11111100	11111101	11111110	11111111

00000000	00000001	00000010	00000011	  ????

00000100	00000101	00000110	00000111	  ????

00001000	00001001	00001010	00001011	  ????

00001100	00001101	00001110	00001111	  ????

00010000	00010001	00010010	00010011	  ????

00010100	00010101	00010110	00010111	  ????

00011000	00011001	00011010	00011011	  ????

00011100	00011101	00011110	00011111	  ????

00100000	00100001	00100010	00100011	  ?!"#

00100100	00100101	00100110	00100111	  $%&'

00101000	00101001	00101010	00101011	  ()*+

00101100	00101101	00101110	00101111	  ,-./

00110000	00110001	00110010	00110011	  0123

00110100	00110101	00110110	00110111	  4567

00111000	00111001	00111010	00111011	  89:;

00111100	00111101	00111110	00111111	  <=>?

01000000	01000001	01000010	01000011	  @ABC

01000100	01000101	01000110	01000111	  DEFG

01001000	01001001	01001010	01001011	  HIJK

01001100	01001101	01001110	01001111	  LMNO

01010000	01010001	01010010	01010011	  PQRS

01010100	01010101	01010110	01010111	  TUVW

01011000	01011001	01011010	01011011	  XYZ[

01011100	01011101	01011110	01011111	  \]^_

01100000	01100001	01100010	01100011	  `abc

01100100	01100101	01100110	01100111	  defg

01101000	01101001	01101010	01101011	  hijk

01101100	01101101	01101110	01101111	  lmno

01110000	01110001	01110010	01110011	  pqrs

01110100	01110101	01110110	01110111	  tuvw

01111000	01111001	01111010	01111011	  xyz{

01111100	01111101	01111110	01111111	  |}~?

10000000	10000001	10000010	10000011	  ????

10000100	10000101	10000110	10000111	  ????

10001000	10001001	10001010	10001011	  ????

10001100	10001101	10001110	10001111	  ????

10010000	10010001	10010010	10010011	  ????

10010100	10010101	10010110	10010111	  ????

10011000	10011001	10011010	10011011	  ????

10011100	10011101	10011110	10011111	  ????

10100000	10100001	10100010	10100011	  ????

10100100	10100101	10100110	10100111	  ????

10101000	10101001	10101010	10101011	  ????

10101100	10101101	10101110	10101111	  ????

10110000	10110001	10110010	10110011	  ????

10110100	10110101	10110110	10110111	  ????

10111000	10111001	10111010	10111011	  ????

10111100	10111101	10111110	10111111	  ????

11000000	11000001	11000010	11000011	  ????

11000100	11000101	11000110	11000111	  ????

11001000	11001001	11001010	11001011	  ????

11001100	11001101	11001110	11001111	  ????

11010000	11010001	11010010	11010011	  ????

11010100	11010101	11010110	11010111	  ????

11011000	11011001	11011010	11011011	  ????

11011100	11011101	11011110	11011111	  ????

11100000	11100001	11100010	11100011	  ????

11100100	11100101	11100110	11100111	  ????

11101000	11101001	11101010	11101011	  ????

11101100	11101101	11101110	11101111	  ????

11110000	11110001	11110010	11110011	  ????

11110100	11110101	11110110	11110111	  ????

11111000	11111001	11111010	11111011	  ????

11111100	11111101	11111110	11111111	  ????

```

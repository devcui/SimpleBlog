---
title: "解析Stcgal"
date: 2021-12-05T11:30:03+00:00
tags: ["stcgal","python","mcu"]
series: []
categories: ["烧录工具"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

# 我没有写过xxx但是今天需求就是涉及到了xxx怎么办

这是我所认为你是否是一名软开人员的基本素质要求，你没接触过但是看看能理解能改改，具备这种素质的，那么你可以说你是一名软开，如果不具备，那么你可能只是一个工具人而已，我将在下文结合自身来进行一次自我检测，看看自己是否有没有这种素质

# 我的方法

看任何东西时，先看脉络流程，在看具体
看不懂没关系，看不懂可以先猜一个大概
比如def abs(file) 你就猜这是一个返回绝对路径的东西
再比如transS2I(s) 可能看不明白，但通过trans猜个转译也行，标注出来，然后往下继续看流程看脉络

当你浏览的差不多以后，在一步一步的进行剖析，去印证你猜的结果就好，猜错了改过来。

# 目录结构

```sh
- debian
- doc
- stcgal
	-  __init__.py
	-  __main__.py
	-  frontend.py
	-  ihex.py
	-  models.py
	-  options.py
	-  protocols.py
	-  utils.py
- tests
- setup.py
- stcgal.py
```

这里我从一级目录的两个py文件看起

# steup.py

见名知意,steup应该和安装有关

```py
// 这里面只有一个函数，提供了name,version,packages,install_requires
// 看到这里，我猜测就是类似npm publish的第三方包，然后npm install xxx就可以安装调用
// 不过据我了解，python的安装是pip应该是 pip install stcgal就可以安装使用了
// 具体参数我百度了一下列出
setup(
	// 包名
    name = "stcgal",
	// 包版本
    version = stcgal.__version__,
	// find_packages包目录，排除了doc和测试包
    packages = find_packages(exclude=["doc", "tests"]),
	// install_requires 安装的前提条件
	// pyserial https://pypi.org/project/pyserial/ 串口库版本要>3.0
	// tqdm https://pypi.org/project/tqdm/ 为循环显示进度条的库? > 4.0
    install_requires = ["pyserial>=3.0", "tqdm>=4.0.0"],
	// 将“extras”(项目的可选特性)的名称映射到字符串或字符串列表，以指定必须安装哪些其他发行版来支持这些特性。嗯，要必须安装pyusb才行
    extras_require = {
        "usb": ["pyusb>=1.0.0"]
    },
	// 这里我猜这是命令的 名称 在terminal环境中
    entry_points = {
        "console_scripts": [
            "stcgal = stcgal.frontend:cli",
        ],
    },
	// 描述和一些其他的东西
    description = "STC MCU ISP flash tool",
    long_description = long_description,
    long_description_content_type = "text/markdown",
    keywords = "stc mcu microcontroller 8051 mcs-51",
    url = "https://github.com/grigorig/stcgal",
    author = "Grigori Goronzy",
    author_email = "greg@kinoho.net",
    license = "MIT License",
    platforms = "any",
// 分类符, 这样便于 pypi 索引, 由 pypi 固定提供, [https://pypi.python.org/pypi?%3Aaction=list_classifiers](https://pypi.python.org/pypi?%3Aaction=list_classifiers)
    classifiers = [
        "Development Status :: 5 - Production/Stable",
        "Environment :: Console",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Operating System :: POSIX",
        "Operating System :: Microsoft :: Windows",
        "Operating System :: MacOS",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.4",
        "Programming Language :: Python :: 3.5",
        "Programming Language :: Python :: 3.6",
        "Topic :: Software Development :: Embedded Systems",
        "Topic :: Software Development",
    ],
	// 测试
    test_suite = "tests",
	// 测试需要的依赖
    tests_require = ["PyYAML"],
)
```

然后我们把endpoint改了试试，去印证一下我的猜测，改成 stcgal1111 然后执行 `python3.7 ./steup.py install`直接安装

![图片.png](https://b3logfile.com/file/2020/09/solofetchupload8885307301536582061-c54cc82c.png)

# stcgal.py

```py
// 闭眼猜这是引包，sys就类似于go的os应该，system
import sys
// 下面是他自己写的包
import stcgal.frontend

if __name__ == "__main__":
    sys.exit(stcgal.frontend.cli())
```

`__name__` 这个写法很奇怪，百度

![图片.png](https://b3logfile.com/file/2020/09/solofetchupload2759356038760349368-15ea8827.png)

如果调用放为__main__那么触发 ` sys.exit(stcgal.frontend.cli())`函数

sys.exit 根据不同的数字判定当前是怎么个退出，是有异常退出呢？是成功退出呢？意外中断呢？以下是翻出来的API

![图片.png](https://b3logfile.com/file/2020/09/solofetchupload6504189598197810183-8e87e7c0.png)

# cli()

```py
// python创建函数方式
def cli():
   // 创建了一个格式化参数对象,传入了argparse.RawDescriptionHelpFormatter,和描述
  // `RawDescriptionHelpFormatter``RawTextHelpFormatter``ArgumentDefaultsHelpFormatter``MetavarTypeHelpFormatter` 一共有这么些转换方式，不细究
    parser = argparse.ArgumentParser(formatter_class=argparse.RawDescriptionHelpFormatter,
                                     description="stcgal {} - an STC MCU ISP flash tool\n".format(stcgal.__version__) +
                                                 "(C) 2014-2018 Grigori Goronzy and others\nhttps://github.com/grigorig/stcgal")
	// 下面是加参数项，有详细的描述，翻译一下就好,arg是区分类型的,传入type,default为不传参数的默认值
    parser.add_argument("code_image", help="code segment file to flash (BIN/HEX)", type=argparse.FileType("rb"), nargs='?')
    parser.add_argument("eeprom_image", help="eeprom segment file to flash (BIN/HEX)", type=argparse.FileType("rb"), nargs='?')
    parser.add_argument("-a", "--autoreset", help="cycle power automatically by asserting DTR", action="store_true")
    parser.add_argument("-r", "--resetcmd",  help="shell command for board power-cycling (instead of DTR assertion)", action="store")
    parser.add_argument("-P", "--protocol", help="protocol version (default: auto)",
                        choices=["stc89", "stc12a", "stc12b", "stc12", "stc15a", "stc15", "stc8", "usb15", "auto"], default="auto")
    parser.add_argument("-p", "--port", help="serial port device", default="/dev/ttyUSB0")
    parser.add_argument("-b", "--baud", help="transfer baud rate (default: 19200)", type=BaudType(), default=19200)
    parser.add_argument("-l", "--handshake", help="handshake baud rate (default: 2400)", type=BaudType(), default=2400)
    parser.add_argument("-o", "--option", help="set option (can be used multiple times, see documentation)", action="append")
    parser.add_argument("-t", "--trim", help="RC oscillator frequency in kHz (STC15+ series only)", type=float, default=0.0)
    parser.add_argument("-D", "--debug", help="enable debug output", action="store_true")
    parser.add_argument("-V", "--version", help="print version info and exit", action="store_true")
    opts = parser.parse_args()

    # run programmer
    gal = StcGal(opts)
    return gal.run()
```

如果还是不清楚，中间做了什么，那就直接print出来不明白的步骤，看打印信息

![图片.png](https://b3logfile.com/file/2020/09/solofetchupload85773580794691002-650ce5f1.png)

比如parser.parse_args()就是按照用户执行命令输入的参数转换成了对象
继续往下走 `StcGal(opts)`

# Stcgal

```py
// python中类的定义
class StcGal:
	// 注释
    """STC ISP flash tool frontend"""
	// 构造函数,self = js中的this,opts为调用时传入的参数
    def __init__(self, opts):
        self.opts = opts
	// 初始化 规则
        self.initialize_protocol(opts)
	// 这里主要就是代理 cli中的-p参数然后分配不同的class给 self.protocol
    def initialize_protocol(self, opts):
        """Initialize protocol backend"""
        if opts.protocol == "stc89":
            self.protocol = Stc89Protocol(opts.port, opts.handshake, opts.baud)
        elif opts.protocol == "stc12a":
            self.protocol = Stc12AProtocol(opts.port, opts.handshake, opts.baud)
        elif opts.protocol == "stc12b":
            self.protocol = Stc12BProtocol(opts.port, opts.handshake, opts.baud)
        elif opts.protocol == "stc12":
            self.protocol = Stc12Protocol(opts.port, opts.handshake, opts.baud)
        elif opts.protocol == "stc15a":
            self.protocol = Stc15AProtocol(opts.port, opts.handshake, opts.baud,
                                           round(opts.trim * 1000))
        elif opts.protocol == "stc15":
            self.protocol = Stc15Protocol(opts.port, opts.handshake, opts.baud,
                                          round(opts.trim * 1000))
        elif opts.protocol == "stc8":
            self.protocol = Stc8Protocol(opts.port, opts.handshake, opts.baud,
                                         round(opts.trim * 1000))
        elif opts.protocol == "usb15":
            self.protocol = StcUsb15Protocol()
        else:
            self.protocol = StcAutoProtocol(opts.port, opts.handshake, opts.baud)
        self.protocol.debug = opts.debug
```

# run()

run函数分为两段
开始猜吧

```
try:
	// 连接
            self.protocol.connect(autoreset=self.opts.autoreset, resetcmd=self.opts.resetcmd)
            if isinstance(self.protocol, StcAutoProtocol):
                if not self.protocol.protocol_name:
                    raise StcProtocolException("cannot detect protocol")
                base_protocol = self.protocol
                self.opts.protocol = self.protocol.protocol_name
                print("Protocol detected: %s" % self.opts.protocol)
                # recreate self.protocol with proper protocol class
                self.initialize_protocol(self.opts)
            else:
                base_protocol = None
	// 调用 分配好的规则对象 对应的 初始化函数生成对象
            self.protocol.initialize(base_protocol)
```

第二部分

```py
try:
            if self.opts.code_image:
		// 刷程序了开始
                self.program_mcu()
		// 正常流程退出
                return 0
	// 关闭连接
            self.protocol.disconnect()
            return 0
```

# mcu()

```py
def program_mcu(self):
        """Execute the standard programming flow."""

        code_size = self.protocol.model.code
        ee_size = self.protocol.model.eeprom

        print("Loading flash: ", end="")
        sys.stdout.flush()
	// 获取文件信息
        bindata = self.load_file_auto(self.opts.code_image)

        # warn if it overflows
	// 代码量超了
        if len(bindata) > code_size:
            print("WARNING: code_image overflows into eeprom segment!", file=sys.stderr)
	// 文件大小超了
        if len(bindata) > (code_size + ee_size):
            print("WARNING: code_image truncated!", file=sys.stderr)
            bindata = bindata[0:code_size + ee_size]

        # add eeprom data if supplied
        if self.opts.eeprom_image:
            print("Loading EEPROM: ", end="")
            sys.stdout.flush()
            eedata = self.load_file_auto(self.opts.eeprom_image)
            if len(eedata) > ee_size:
                print("WARNING: eeprom_image truncated!", file=sys.stderr)
                eedata = eedata[0:ee_size]
            if len(bindata) < code_size:
                bindata += bytes(code_size - len(bindata))
            elif len(bindata) > code_size:
                print("WARNING: eeprom_image overlaps code_image!", file=sys.stderr)
                bindata = bindata[0:code_size]
            bindata += eedata

        # pad to 512 byte boundary
        if len(bindata) % 512:
            bindata += b'\xff' * (512 - len(bindata) % 512)

        if self.opts.option: self.emit_options(self.opts.option)
	// 握手检测
        self.protocol.handshake()
	// 清除数据
        self.protocol.erase_flash(len(bindata), code_size)
	// 写数据
        self.protocol.program_flash(bindata)
        self.protocol.program_options()
        self.protocol.disconnect()
```

# erase_flash(len(bindata), code_size)

```py
def erase_flash(self, erase_size, _):
        """Erase the MCU's flash memory.

        	是用块儿擦除命令擦除闪存
        """

        blks = ((erase_size + 511) // 512) * 2
        print("Erasing %d blocks: " % blks, end="")
        sys.stdout.flush()
	// 数据包
        packet = bytes([0x84, blks, 0x33, 0x33, 0x33, 0x33, 0x33, 0x33])
	// 发送数据包
        self.write_packet(packet)
	// 接受返回信息
        response = self.read_packet()
        if response[0] != 0x80:
            raise StcProtocolException("incorrect magic in erase packet")
        print("done")
```

```py
def write_packet(self, packet_data):
        """Send packet to MCU.

        用有效的负载构造数据包发送到MCU
        """

        # frame start and direction magic
        packet = bytes()
        packet += self.PACKET_START
        packet += self.PACKET_HOST

        # packet length and payload
        packet += struct.pack(">H", len(packet_data) + 5)
        packet += packet_data

        # checksum and end code
        packet += bytes([sum(packet[2:]) & 0xff])
        packet += self.PACKET_END

        self.dump_packet(packet, receive=False)
        self.ser.write(packet)
        self.ser.flush()
```

这里ser是设备信息，之前在connect时初始化完毕了

```py
def connect(self, autoreset=False, resetcmd=False):
        """Connect to MCU and initialize communication.

        Set up serial port, send sync sequence and get part info.
        """

        self.ser = serial.Serial(port=self.port, parity=self.PARITY)
        # set baudrate separately to work around a bug with the CH340 driver
        # on older Linux kernels
        self.ser.baudrate = self.baud_handshake

        # fast timeout values to deal with detection errors
        self.ser.timeout = 0.5
        self.ser.interCharTimeout = 0.5

        # avoid glitches if there is something in the input buffer
        self.ser.flushInput()

        if autoreset:
            self.reset_device(resetcmd)
        else:
            print("Waiting for MCU, please cycle power: ", end="")
            sys.stdout.flush()

        # send sync, and wait for MCU response
        # ignore errors until we see a valid response
        self.status_packet = None
        while not self.status_packet:
            try:
                self.pulse()
                self.status_packet = self.get_status_packet()
                if len(self.status_packet) < 23:
                    raise StcProtocolException("status packet too short")
            except (StcFramingException, serial.SerialTimeoutException): pass
        print("done")

        # conservative timeout values
        self.ser.timeout = 15.0
        self.ser.interCharTimeout = 1.0
```

# program_flash(bindata)

```py
def program_flash(self, data):
        """Program the MCU's flash memory.

     	写到MCU,块大小取决于MCU的RAM大小  PROGRAM_BLOCKSIZE
        """
	// 循环data，步进 PROGRAM_BLOCKSIZE
        for i in range(0, len(data), self.PROGRAM_BLOCKSIZE):
		// 构造数据包
            packet = bytes(3)
            packet += struct.pack(">H", i)
            packet += struct.pack(">H", self.PROGRAM_BLOCKSIZE)
            packet += data[i:i+self.PROGRAM_BLOCKSIZE]
            while len(packet) < self.PROGRAM_BLOCKSIZE + 7: packet += b"\x00"
            csum = sum(packet[7:]) & 0xff
	// 发送数据包
            self.write_packet(packet)
	// 接受返回信息
            response = self.read_packet()
            if len(response) < 1 or response[0] != 0x80:
                raise StcProtocolException("incorrect magic in write packet")
            elif len(response) < 2 or response[1] != csum:
                raise StcProtocolException("verification checksum mismatch")
	// 每次写完都记录一下当前位置，块内容，数据量 暂时不知道干啥的
            self.progress_cb(i, self.PROGRAM_BLOCKSIZE, len(data))
        self.progress_cb(len(data), self.PROGRAM_BLOCKSIZE, len(data))
```
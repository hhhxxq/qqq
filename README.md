graph TD
    %% 定义样式
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    classDef key fill:#f9f,stroke:#f6f,stroke-width:2px,stroke-dasharray: 5 5;
    classDef chaos fill:#bbf,stroke:#33f,stroke-width:2px;
    classDef process fill:#dfd,stroke:#393,stroke-width:1px;
    classDef feedback fill:#fbb,stroke:#f33,stroke-width:1px;

    %% 外部输入
    InputImage[("明文图像 P<br>(M × N × 3)")]:::plain
    UserKey[("用户密钥 K<br>(256-bit)")]:::key

    %% —————— 第一部分：密钥生成与混沌控制域 ——————
    subgraph SubKey "【I】密钥生成与混沌控制域 (KGen & Control)"
        direction TB
        SHA256[("SHA-256 哈希计算")]:::process
        KeyMixer("密钥融合算子<br>(K XOR Hash)"):::process
        
        ParamInit("系统参数 & 初值生成<br>(x₀, y₀, z₀, w₀, v₀, a, b, c...)"):::process
        
        CAGen["CA (元胞自动机)<br>演化引擎 (如 Rule 30)"]:::chaos
        
        subgraph SubChaos "核心混沌系统"
            Lorenz5D["5D Lorenz 混沌系统<br>ẋ = a(y-x) + ..."]:::chaos
        end
        
        CARule("CA 参数扰动算子<br>(p' = p + ΔCA)"):::chaos

        %% 连接
        InputImage --> SHA256
        SHA256 --> KeyMixer
        UserKey --> KeyMixer
        KeyMixer --> ParamInit
        ParamInit --> Lorenz5D
        ParamInit --> CAGen
        CAGen --"扰动信号"--> CARule
        CARule --"动态参数"--> Lorenz5D
    end

    %% —————— 第二部分：核心加密迭代域 ——————
    subgraph SubEncrypt "【II】核心加密迭代域 (Encryption Core)"
        direction TB
        SeqGen("混沌序列提取与处理<br>(Quantization & Sampling)"):::chaos
        
        ImagePre("图像预处理<br>(RGB分解 / 拍平)"):::process

        subgraph SubScramble "级联置乱 (Confusion)"
            direction LR
            PixelScramble("像素级置乱<br>(如 Spiral/Scan)"):::process
            BitSplit("位平面分解<br>(8 Bit Planes)"):::process
            BitScramble("位级置乱<br>(如 Zigzag)"):::process
            BitCombine("位平面重组"):::process
        end

        subgraph SubDiffuse "动态扩散 (Diffusion)"
            direction LR
            DSBox("混沌动态 S 盒<br>(Substitution)"):::process
            XORAction("异或操作 (XOR)"):::process
            PixelFeed("像素反馈网络"):::process
        end

        %% 连接
        InputImage --> ImagePre
        Lorenz5D --"连续混沌流"--> SeqGen
        
        ImagePre --> PixelScramble
        SeqGen --"位置地址序列"--> PixelScramble
        PixelScramble --> BitSplit
        BitSplit --> BitScramble
        SeqGen --"位异或序列"--> BitScramble
        BitScramble --> BitCombine
        
        BitCombine --> DSBox
        SeqGen --"S盒控制序列"--> DSBox
        DSBox --> XORAction
        SeqGen --"扩散密钥流"--> XORAction
        XORAction --> PixelFeed
        PixelFeed --"上一个密文像素"--> XORAction
    end

    %% —————— 第三部分：输出与反馈机制 ——————
    OutputImage[("密文图像 C<br>(M × N × 3)")]:::plain

    subgraph SubRobust "【III】鲁棒性与锚点机制"
        direction TB
        AnchorGen("锚点生成器<br>(混沌状态快照)"):::feedback
        RobustFeed("锚点反馈通道"):::feedback
    end

    %% 总体连接
    PixelFeed --> OutputImage
    
    %% 反馈回路
    OutputImage --"关键像素采样"--> AnchorGen
    AnchorGen --"同步/校正信号"--> Lorenz5D
    XORAction -.-> RobustFeed .-> DSBox

    %% 样式应用
    class SubKey,SubChaos,SubEncrypt,SubScramble,SubDiffuse,SubRobust fill:#f9f9f9,stroke:#999,stroke-width:1px,stroke-dasharray: 5 5;

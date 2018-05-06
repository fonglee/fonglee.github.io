---
layout: post
title: webcam代码分析
excerpt: "webcam代码分析"
modified: 2014-10-04
tags: [c]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

# capture_open

    void *capture = capture_open("/dev/video0", VIDEO_WIDTH, VIDEO_HEIGHT, AV_PIX_FMT_NV12);

    return ctx;

    struct Buffer
    {
        void *start;
        size_t length;
    };
    typedef struct Buffer Buffer;

    struct Ctx
    {
        int vid;//ctx->vid = id; int id = open(dev_name, O_RDWR);
        int width, height;  // 输出图像大小   ctx->width = t_width;ctx->height = t_height;
        struct SwsContext *sws; // 用于转换     ctx->sws = sws_getContext(fmt.fmt.pix.width, fmt.fmt.pix.height, pixfmt,ctx->width, ctx->height, tarfmt,SWS_FAST_BILINEAR, 0, 0, 0);
        int rows;   // 用于 sws_scale() ctx->rows = fmt.fmt.pix.height;
        int bytesperrow; // 用于cp到 pic_src ctx->bytesperrow = fmt.fmt.pix.bytesperline;
        AVPicture pic_src, pic_target;  // 用于 sws_scale avpicture_alloc(&ctx->pic_target, tarfmt, ctx->width, ctx->height);
        Buffer bufs[2];     // 用于 mmap 
        PixelFormat fmt;//ctx->fmt = tarfmt;
    };
    typedef struct Ctx Ctx;

    /*参数说明：参数类型为V4L2的能力描述类型struct v4l2_capability ；
    返回值说明： 执行成功时，函数返回值为 0；函数执行成功后，struct v4l2_capability 结构体变量中的返回当前视频设备所支持的功能；例如支持视频捕获功能V4L2_CAP_VIDEO_CAPTURE、V4L2_CAP_STREAMING等。*/
    ioctl(id, VIDIOC_QUERYCAP, &caps); // to query caps 查询设备属性

    bufs.count = 2;
    bufs.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
    bufs.memory = V4L2_MEMORY_MMAP;

    ioctl(id, VIDIOC_REQBUFS, &bufs) //v4l2_requestbuffers  bufs 请求V4L2驱动分配视频缓冲区（申请V4L2视频驱动分配内存），V4L2是视频设备的驱动层，位于内核空间，所以通过VIDIOC_REQBUFS控制命令字申请的内存位于内核空间，应用程序不能直接访问，需要通过调用mmap内存映射函数把内核空间内存映射到用户空间后，应用程序通过访问用户空间地址来访问内核空间。

    v4l2_buffer buf;
    buf.type = bufs.type;
    buf.memory = bufs.memory;
    ioctl(id, VIDIOC_QUERYBUF, &buf);
    /*功能： 查询已经分配的V4L2的视频缓冲区的相关信息，包括视频缓冲区的使用状态、在内核空间的偏移地址、缓冲区长度等。在应用程序设计中通过调VIDIOC_QUERYBUF来获取内核空间的视频缓冲区信息，然后调用函数mmap把内核空间地址映射到用户空间，这样应用程序才能够访问位于内核空间的视频缓冲区。
    参数说明：参数类型为V4L2缓冲区数据结构类型    struct v4l2_buffer  ；
    返回值说明： 执行成功时，函数返回值为 0；struct v4l2_buffer结构体变量中保存了指令的缓冲区的相关信息；
    一般情况下，应用程序中调用VIDIOC_QUERYBUF取得了内核缓冲区信息后，紧接着调用mmap函数把内核空间地址映射到用户空间，方便用户空间应用程序的访问。*/

    //查询并显示所有支持的格式
    /*参数说明：参数类型为V4L2的视频格式描述符类型 struct v4l2_fmtdesc
    返回值说明： 执行成功时，函数返回值为 0；struct v4l2_fmtdesc 结构体中的 .pixelformat和 .description 成员返回当前视频设备所支持的视频格式；*/
    ioctl(id, VIDIOC_ENUM_FMT, &fmt_desc);
    //读取当前驱动的帧捕获格式
    rc = ioctl(id, VIDIOC_G_FMT, &fmt);

    fprintf(stderr, "capture_width=%d, height=%d, stride=%d\n", fmt.fmt.pix.width, fmt.fmt.pix.height,
            fmt.fmt.pix.bytesperline);

    ctx->width = t_width;
    ctx->height = t_height;
    ctx->sws = sws_getContext(fmt.fmt.pix.width, fmt.fmt.pix.height, pixfmt,ctx
    ->width, ctx->height, tarfmt,
    SWS_FAST_BILINEAR, 0, 0, 0);

    ctx->rows = fmt.fmt.pix.height;
    ctx->bytesperrow = fmt.fmt.pix.bytesperline;
    avpicture_alloc(&ctx->pic_target, tarfmt, ctx->width, ctx->height);//tarfmt输入是函数的帧格式，pic_target是AVPicture结构体变量

    /*
    功能： 投放一个空的视频缓冲区到视频缓冲区输入队列中 ；
    参数说明：参数类型为V4L2缓冲区数据结构类型    struct v4l2_buffer ；
    返回值说明： 执行成功时，函数返回值为 0；函数执行成功后，指令(指定)的视频缓冲区进入视频输入队列，在启动视频设备拍摄图像时，相应的视频数据被保存到视频输入队列相应的视频缓冲区中。
    */
    ioctl(id, VIDIOC_QBUF, &buf)；

    /*
    功能： 启动视频采集命令，应用程序调用VIDIOC_STREAMON启动视频采集命令后，视频设备驱动程序开始采集视频数据，并把采集到的视频数据保存到视频驱动的视频缓冲区中。
    参数说明：参数类型为V4L2的视频缓冲区类型 enum v4l2_buf_type ；
    返回值说明： 执行成功时，函数返回值为 0；函数执行成功后，视频设备驱动程序开始采集视频数据，此时应用程序一般通过调用select函数来判断一帧视频数据是否采集完成，当视频设备驱动完成一帧视频数据采集并保存到视频缓冲区中时，select函数返回，应用程序接着可以读取视频数据；否则select函数阻塞直到视频数据采集完成。Select函数的使用请读者参考相关资料。
    */
    ioctl(id, VIDIOC_STREAMON, &type);

# vc_open
    void *encoder = vc_open(VIDEO_WIDTH, VIDEO_HEIGHT, VIDEO_FPS);

    struct Ctx
    {
        x264_t *x264;
        x264_picture_t picture;
        x264_param_t param;
        void *output;       // 用于保存编码后的完整帧
        int output_bufsize, output_datasize;
        int64_t pts;        // 输入 pts
        int64_t (*get_pts)(struct Ctx *);

        int64_t info_pts, info_dts;
        int info_key_frame;
        int info_valid;

        int force_keyframe;
    };

    ctx->param.i_width = width;//视频属性
    ctx->param.i_height = height;//视频属性
    ctx->param.b_repeat_headers = 1;  // 重复SPS/PPS 放到关键帧前面
    ctx->param.b_cabac = 1;//数据流参数
    ctx->param.i_threads = 2;/* encode multiple frames in parallel 平行的编码多帧*/

    ctx->param.i_fps_num = (int)fps; //每秒帧数
    ctx->param.i_fps_den = 1;
    ctx->param.i_keyint_max = ctx->param.i_fps_num * 2;/* Force an IDR keyframe at this interval  在间隔中强制IDR关键字*/
    ctx->param.rc.i_bitrate = 50;

    ctx->x264 = x264_encoder_open(&ctx->param);

    x264_picture_init(&ctx->picture);
    ctx->picture.img.i_csp = X264_CSP_I420; /* Colorspace */
    ctx->picture.img.i_plane = 3; /* Number of image planes */

    ctx->output = malloc(128*1024);
    ctx->output_bufsize = 128*1024;
    ctx->output_datasize = 0;

    ctx->get_pts = first_pts;
    /*
    static int64_t first_pts (struct Ctx *ctx)
    {
        ctx->pts = curr();
        ctx->get_pts = next_pts;
        return 0;
    }
    static int64_t curr ()
    {
        struct timeval tv;
        gettimeofday(&tv, 0);//获得当前精确时间（1970年1月1日到现在的时间）
        double n = tv.tv_sec + 0.000001*tv.tv_usec;
        n *= 90000.0;
        return (int64_t)n;
    }

    static int64_t next_pts (struct Ctx *ctx)
    {
        int64_t now = curr();
        return now - ctx->pts;
    }
    */


    ctx->info_valid = 0;


# sender_open

    void *sender = sender_open(TARGET_IP, TARGET_PORT);

# capture\_get\_picture
    
    capture_get_picture(capture, &pic);

    ioctl(ctx->vid, VIDIOC_DQBUF, &buf)

    ctx->pic_src.data[0] = (unsigned char*)ctx->bufs[buf.index].start;
    ctx->pic_src.data[1] = ctx->pic_src.data[2] = ctx->pic_src.data[3] = 0;
    ctx->pic_src.linesize[0] = ctx->bytesperrow;
    ctx->pic_src.linesize[1] = ctx->pic_src.linesize[2] = ctx->pic_src.linesize[3] = 0;


    int rs = sws_scale(ctx->sws, ctx->pic_src.data, ctx->pic_src.linesize,
            0, ctx->rows, ctx->pic_target.data, ctx->pic_target.linesize);

    for (int i = 0; i < 4; i++) {
            pic->data[i] = ctx->pic_target.data[i];
            pic->stride[i] = ctx->pic_target.linesize[i];
        }

    ioctl(ctx->vid, VIDIOC_QBUF, &buf)

# vc_compress  

    int rc = vc_compress(encoder, pic.data, pic.stride, &outdata, &outlen);

    for (int i = 0; i < 4; i++) {
        c->picture.img.plane[i] = data[i];
        c->picture.img.i_stride[i] = stride[i];
    }

    x264_picture_t *pic = &c->picture;

    c->picture.i_pts = c->get_pts(c);//获得时间戳

    int rc = x264_encoder_encode(c->x264, &nals, &nal_cnt, pic, &pic_out);//输入 c->x264，pic，输出nal，nal_cnt,pic_out

    dumpnals(pic_out.i_type, nals, nal_cnt);
    /*
    static void dumpnals (int type, x264_nal_t *nal, int nals)
    {
        fprintf(stderr, "======= dump nals %d type=%d, ======\n", nals, type);
        for (int i = 0; i < nals; i++) {
            fprintf(stderr, "\t nal  #%d: %dbytes, nal type=%d ", i, nal[i].i_payload, nal[i].i_type);
            dumpnal(&nal[i]);
            fprintf(stderr, "\n");
        }
    }

    static void dumpnal (x264_nal_t *nal)
    {
        // 打印前面 10 个字节
        for (int i = 0; i < nal->i_payload && i < 20; i++) {
            fprintf(stderr, "%02x ", (unsigned char)nal->p_payload[i]);
        }
    }
    */

    encode_nals(c, nals, nal_cnt);
    /*
        static int encode_nals (Ctx *c, x264_nal_t *nals, int nal_cnt)
    {
        char *pout = (char*)c->output;//128*1024的大小
        c->output_datasize = 0;
        for (int i = 0; i < nal_cnt; i++) {
            if (c->output_datasize + nals[i].i_payload > c->output_bufsize) {
                // 扩展
                c->output_bufsize = (c->output_datasize+nals[i].i_payload+4095)/4096*4096;
                c->output = realloc(c->output, c->output_bufsize);
            }
            memcpy(pout+c->output_datasize, nals[i].p_payload, nals[i].i_payload);
            c->output_datasize += nals[i].i_payload;
        }

        return c->output_datasize;
    }
    */








import os
import glob
import argparse
from PyPDF2 import PdfReader, PdfWriter
import PyPDF2
from reportlab.lib.units import cm
from reportlab.pdfgen import canvas
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.pdfbase import pdfmetrics
from tqdm import tqdm
import pikepdf

pdfmetrics.registerFont(TTFont('SimSun', './fonts/SimSun.ttf'))  # 注册字体

last_width = 0
last_height = 0

def create_watermark(content, width, height, args):
    # 默认大小为21cm*29.7cm
    file_name = "mark.pdf"
    # 比例，用于自适应 pdf 页面大小
    width = float(width) * 0.0352
    height = float(height) * 0.0352
    ratio_w = width / 21
    ratio_h = height / 29.7
    c = canvas.Canvas(file_name, pagesize=(width * cm, height * cm))
    # 移动坐标原点(坐标系左下为(0,0))
    c.translate(10 * cm * ratio_w, 5 * cm * ratio_h)

    # 设置字体，默认采用 SimSun 字体
    c.setFont(args.font, args.size * (ratio_w + ratio_h) / 2)
    # 指定描边的颜色
    c.setStrokeColorRGB(0, 1, 0)
    # 默认旋转30度，坐标系被旋转
    c.rotate(args.angle)
    # 指定填充颜色
    c.setFillColorRGB(0, 0, 0, args.alpha)

    # 画几个文本，注意坐标系旋转的影响
    for i in range(5):
        for j in range(10):
            # 使用 比例 自适应水印文字位置
            a = 10 * (i - 1) * ratio_w
            b = 5 * (j - 2) * ratio_h
            c.drawString(a * cm, b * cm, content)
    # 关闭并保存pdf文件
    c.save()
    return file_name

def add_watermark(pdf_file_in, pdf_file_out,password, UserPassword,args):
    global last_height, last_width
    pdf_watermark = []

    pdf_output = PdfWriter()
    input_stream = open(pdf_file_in, 'rb')
    pdf_input = PdfReader(input_stream, strict=False)

    # 获取PDF文件的页数
    pageNum = len(pdf_input.pages)

    # 给每一页打水印
    for i in range(pageNum):
        page = pdf_input.pages[i]
        # 获取当前页面实际宽高
        width = pdf_input.pages[i].mediabox.width
        height = pdf_input.pages[i].mediabox.height
        if width != last_width or height != last_height:
            pdf_file_mark = create_watermark(watermark_text, width, height, args)  # 生成水印文件

            pdf_watermark = PdfReader(open(pdf_file_mark, 'rb'), strict=False)  # 读入水印pdf文件
            last_width = width
            last_height = height
        page.merge_page(pdf_watermark.pages[0])
        page.compress_content_streams()  # 压缩内容
        pdf_output.add_page(page)
        
    #permissions =  PyPDF2.PdfFileWriter.ALLOW_PRINTING | PyPDF2.PdfFileWriter.ALLOW_COPY
    pdf_output.encrypt(user_password =password,owner_password = UserPassword, use_128bit = True,permissions_flag=0b0100)  # 设置密码权限保护

    pdf_output.write(open(pdf_file_out, 'wb'))
    


  

def process_folder(folder_path, watermark_text,password, UserPassword,args):
    for root, dirs, files in os.walk(folder_path):
        for file in files:
             for file in tqdm(files, desc="处理文件夹"):
                if file.endswith(".pdf"):
                    pdf_file_in = os.path.join(root, file)
                    pdf_file_out = pdf_file_in.replace('.pdf', '') + '（添加水印）.pdf'
                    add_watermark(pdf_file_in, pdf_file_out,password, UserPassword,args)
                    print("添加水印成功")
                    print("文件路径为： {}".format(pdf_file_out.replace('/', '\\')))
                
                
                
if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='WaterMarker of argparse')
    parser.add_argument('--text', default='watermark', help="Text to add watermark")
    parser.add_argument('-F', '--file', default='', help="The path to the file to add the watermark to")
    parser.add_argument('--font', default='SimSun', help="Font used for watermark text")
    parser.add_argument('--size', type=int, default=30, help="Font size used for watermark text, defaults to 30, "
                                                             "the size will adjust itself as the page changes")
    parser.add_argument('--alpha', type=float, default=0.1, help="Transparency of watermark text, between 0.0 and 1.0")
    parser.add_argument('--angle', type=int, default=30, help="Rotate the canvas by the angle theta (in degrees)")
    parser.add_argument('-O', '--output', default='', help="File output path after adding watermark (including the "
                                                           "file name), the default is the original file directory")
    parser.add_argument('--password', default='', help="Set password for the generated PDF")
    parser.add_argument('--UserPassword', default='', help="Set UserPassword for the generated PDF")
    args = parser.parse_args()

    text = ''
    if args.text == 'watermark':
        text = input("请输入水印文字：")
    watermark_text = args.text if text == '' else text

    if args.file == '':
        pdf_file_in = input("请输入文件路径：").strip('"').strip(' ')
    else:
        pdf_file_in = args.file
    if args.password =='':
        password = input("请输入文件开启密码：")
    else:
        password = args.password
        
    if args.UserPassword == '':
        UserPassword = input("请输入文件管理员权限密码")
    else:
        UserPassword = args.UserPassword
    
    if os.path.isfile(pdf_file_in):
        pdf_file_out = pdf_file_in.replace('.pdf', '') + '（添加水印）.pdf'
        add_watermark(pdf_file_in, pdf_file_out, password,UserPassword,args)
        print("添加水印成功")
        print("文件路径为： {}".format(pdf_file_out.replace('/', '\\')))
    elif os.path.isdir(pdf_file_in):
        process_folder(pdf_file_in, watermark_text, password,UserPassword,args)
    else:
        print("输入文件或文件夹路径不正确！！!")

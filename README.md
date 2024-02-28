import os
from tkinter import *
from tkinter import filedialog
from PIL import Image

def fun(n):
    n = "0" * (8 - len(str(n))) + str(n)
    t1 = tuple(int(n[i]) for i in range(4))
    t2 = tuple(int(n[i]) for i in range(4, 8))
    return t1, t2

def bun(t1, t2):
    # Convert the tuple back to string and then to integer
    try:
        literal = ''.join(str(x) for x in (t1 + t2))
        n = int(literal) if literal else 0
    except ValueError as e:
        print(f"Error converting to int: {e}, t1: {t1}, t2: {t2}")
        n = 0
    return n

def bin_8(n):
    b = '{0:b}'.format(n)
    return '0' * (8 - len(b)) + b

def Encrypt(directory, name, ext, text):
    img_path = os.path.join(directory, name + ext)
    img = Image.open(img_path)
    img = img.convert("RGBA")
    pixels = img.load()
    width, height = img.size
    l = len(text)
    t1, t2 = fun(l)
    ascii_vals = [ord(i) for i in text]
    pixels[width - 1, height - 1] = t2
    pixels[width - 2, height - 2] = t1
    x, y = 0, 0
    for i in ascii_vals:
        tup = img.getpixel((x, y))
        bn = bin_8(i)
        tup2 = tuple(bin_8(tup[j]) for j in range(4))
        tup3 = tuple(tup2[j][:6] + bn[j*2:(j+1)*2] for j in range(4))
        tup4 = tuple(int(tup3[j], 2) for j in range(4))
        pixels[x, y] = tup4
        x = (x + 1) % width
        if x == 0:
            y += 1
    encrypted_image_path = os.path.join(directory, name + "_e" + ext)
    img.save(encrypted_image_path)
    return encrypted_image_path

def Decrypt(directory, name, ext):
    img_path = os.path.join(directory, name + ext)
    img = Image.open(img_path)
    width, height = img.size
    text = ""
    t2 = img.getpixel((width - 1, height - 1))
    t1 = img.getpixel((width - 2, height - 2))
    l = bun(t1, t2)
    x, y = 0, 0
    for i in range(l):
        tup = img.getpixel((x, y))
        tup1 = tuple(bin_8(tup[j]) for j in range(4))
        c = ''.join(tup1[j][6:] for j in range(4))
        text += chr(int(c, 2))
        x = (x + 1) % width
        if x == 0:
            y += 1
    return text

def fildia():
    f_name = filedialog.askopenfilename(initialdir="/", title="Select an Image File", filetype=(("png files", ".png"), ("jpeg files", ".jpg")))
    sv.set(f_name)

def en():
    text = e1.get("1.0", END).strip()
    e1.delete('1.0', END)
    full_path = sv.get()
    if not full_path:
        l1.config(text="No file selected")
        return
    if os.path.isfile(full_path):
        directory, filename = os.path.split(full_path)
        name, ext = os.path.splitext(filename)
        if not ext:
            l1.config(text="File does not have an extension")
            return
        try:
            encrypted_image_path = Encrypt(directory, name, ext, text)
            l1.config(text=f"Success. Encrypted image saved as: {encrypted_image_path}")
        except Exception as e:
            l1.config(text=f"Failed: {str(e)}")
    else:
        l1.config(text="Selected file not found")

def de():
    e1.delete('1.0', END)
    full_path = sv.get()
    if not full_path:
        l1.config(text="No file selected")
        return
    if os.path.isfile(full_path):
        directory, filename = os.path.split(full_path)
        name, ext = os.path.splitext(filename)
        try:
            text = Decrypt(directory, name, ext)
            e1.insert('1.0', text)
            l1.config(text="Success")
        except Exception as e:
            l1.config(text=f"Failed: {str(e)}")
    else:
        l1.config(text="Selected file not found")

root = Tk()
root.title("Steganography")
l = Label(root, text="Steganography", font="Calibri 20 bold")
lf1 = LabelFrame(root, text="Open an Image")
sv = StringVar()
n_e = Entry(lf1, font="Calibri 12", width=70, textvariable=sv, state=DISABLED)
b = Button(lf1, text="Browse for an Image", command=fildia, font="Calibri 12")
f = Frame(root)
eb = Button(f, text="Encrypt", font="Calibri 17", command=en)
db = Button(f, text="Decrypt", font="Calibri 17", command=de)
lf2 = LabelFrame(root, text="Text Input or Output")
f1 = Frame(lf2)
l1 = Label(root, text="", font="Calibri 17")
s1 = Scrollbar(f1)
e1 = Text(f1, height=10, font="Calibri 13")
s1.pack(side=RIGHT, fill=Y)
e1.pack(side=LEFT, fill=BOTH)
s1.config(command=e1.yview)
e1.config(yscrollcommand=s1.set)
l.pack(padx=10, pady=7)
lf1.pack(padx=10, pady=4, fill=BOTH)
n_e.pack(side=LEFT, fill=BOTH, padx=10, pady=5)
b.pack(side=LEFT, fill=BOTH, padx=10, pady=5)
f.pack(padx=10, pady=4)
eb.pack(padx=10, pady=7, fill=BOTH, side=LEFT)
db.pack(padx=10, pady=7, fill=BOTH, side=LEFT)
f1.pack(padx=10, pady=10)
lf2.pack(padx=10, pady=10)
l1.pack(pady=7)
root.mainloop()

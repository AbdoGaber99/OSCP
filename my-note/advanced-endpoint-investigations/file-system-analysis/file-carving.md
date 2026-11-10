# file carving

{% hint style="success" %}
**Tools**: `okteta`, `dd` , `exiftool` , `binwalk` , `foremost` , `scalpel`
{% endhint %}

### File Headers and Footers

| File Type | Header (Magic Bytes)                                     | Footer / End Marker                        | Notes                                          |
| --------- | -------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| **JPEG**  | FF D8 FF E0 (commonly FF D8 FF)                          | FF D9                                      | Commonly used image format                     |
| **PNG**   | 89 50 4E 47 0D 0A 1A 0A                                  | 49 45 4E 44 AE 42 60 82                    | Uses chunks to mark start/end                  |
| **PDF**   | %PDF (25 50 44 46)                                       | %%EOF (25 25 45 4F 46)                     | Portable Document Format                       |
| **DOCX**  | 50 4B 03 04 (ZIP signature)                              | 50 4B 05 06 (ZIP end of central directory) | DOCX is essentially a ZIP archive of XML files |
| **GIF**   | 47 49 46 38 39 61 (GIF89a) or 47 49 46 38 37 61 (GIF87a) | 00 3B                                      | GIF ends with a semicolon                      |
| **ZIP**   | 50 4B 03 04                                              | 50 4B 05 06                                | General-purpose archive format                 |

### Metadata

**Metadata** is data that provides information about other data. In the context of files, metadata describes attributes such as creation date, modification date, permissions, and more.

**Types of Metadata**

1. **Embedded Metadata:** The information is stored within the file itself. For example, EXIF data in JPEG images includes camera settings, location, and timestamps.
2. **Extended Metadata:** The operating system stores this information as file attributes, such as file permissions and last accessed dates.
3. **External Metadata:** Information here can be managed separately from the file, like log entries or database records tracking file access.

### Manual Carving

### Scenario 1: USB Drive Wipe Recovery

* The first step is to determine the header and footer offsets; As an example, we can assume there is a file at the starting offset **0134483968** and **0134484431** as the ending offset.
* Subtract the offset values, with the ending offset as the minuend. The result is the file size in bytes: `134484431(Ending) - 134483968 (Starting) = 463 bytes.`
* Run `dd` command to extract the file

```bash
dd if=Challenge1_Manual_Carve_usb.img of=Image.png bs=1 skip=134483968 count=463
```

### Scenario 2: Recovering Files from Slack Space

```bash
binwalk Challenge2_slack_space.img 
```

```bash
binwalk -e Challenge2_slack_space.img 
```

### Automatic Carving

### Scenario 3: Recovering Deleted Files with Foremost

```bash
foremost -i Challenge3_deleted_disk.img -o Challenge3_files -c /etc/custom_foremost.conf
```

### A Look at Scalpel

```bash
scalpel Challenge3_deleted_disk.img -o ScalpelOutput -c /etc/scalpel/scalpel.conf
```

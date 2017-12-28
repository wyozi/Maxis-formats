Structure of the packed-BMP formats used for texture storage

## Introduction

Maxis did something weird and crammed a bunch of BMP files together into a single "archive" format.  All the textures are in these four files:

  * sim3d.bmp
  * sky.bmp
  * skydark.bmp
  * tiled1.bmp

skycool.bmp and skygray.bmp are red herrings: they are not referenced anywhere within SimCopter.  But they do look cool.  Maybe there was a planned "choose your fogginess" option where these would have fit into a menu somewhere (View Depth or something).

## Details

*All integers are Little-Endian (Intel) unless otherwise noted*

*Palette data is stored in the CMAP block of .max files: 768 bytes R,G,B.  Make sure to convert to BGR0 if writing a BMP*

```
Structure:
===HEADER===
int32: Total file size
int32: Always 0
int32: Number of images stored in the File
int32: Number of Resolution blocks to follow

RESOLUTIONS (N times, see above):
int32: x
int32: y
int32: attrib (name, int32, or 0)

===TEXTURE===
int32: X Dimension
int32: Y Dimension of Images
int32: Always 0
Y int32s: offset of start of each Row in subsequent data
IMAGE DATA (bytes), each image is X by Y bytes
```

## Sample Code

Here is a Perl script that will take a 768-byte palette as arg0 and a packed-bmp as arg1, and spit out (valid) BMP files with attached palette, 1 per texture in original file.

```perl
#!/usr/bin/perl
use strict;

sub r_v
{
  my $fp = shift;
  my $buf;
  my $read_back = read ($fp, $buf, 4);
  if ($read_back != 4) { die "Short read on input: expected 4 bytes, got $read_back\n"; }
  return unpack ('V', $buf);
}

my $fp;

my @cmap;

my @x;
my @y;
my @tag;

my $temp;

print "Reading colormap.";
open ($fp, $ARGV[0]) or die "can't open $ARGV[0]: $!\n";
binmode($fp);

for (my $i = 0; $i < 256; $i ++)
{
  read ($fp, $temp, 1);
  $cmap[$i][2] = $temp;
  read ($fp, $temp, 1);
  $cmap[$i][1] = $temp;
  read ($fp, $temp, 1);
  $cmap[$i][0] = $temp;
}

close($fp);

print "All done: now opening packed bmp\n";
open ($fp, $ARGV[1]) or die "can't open $ARGV[1]: $!\n";
binmode($fp);

$temp = r_v($fp);
print "Size: $temp\n";
$temp = r_v($fp);
print "Expect 0: $temp\n";
my $num_images = r_v($fp);
print "** $num_images textures in file**\n";
$temp = r_v($fp);
print "Looping to pick up $temp meta-attribs\n";
for (my $i = 0; $i < $num_images; $i ++)
{
  $x[$i] = r_v($fp);
  $y[$i] = r_v($fp);
  $tag[$i] = r_v($fp);
}
print "OK!  Beginning image extract.\n";
for (my $i = 0; $i < $num_images; $i ++)
{
  print ". image " . $i . "\n";
  my $local_x = r_v($fp);
  if ($x[$i] != $local_x)
  {
    print "WARNING: Mismatch between Image Width: $x[$i] vs $local_x\n";
  }
  my $local_y = r_v($fp);
  if ($y[$i] != $local_y)
  {
    print "WARNING: Mismatch between Image Width: $y[$i] vs $local_y\n";
  }
  $temp = r_v($fp);
  print "Expect 0: $temp\n";

  my $offset = 0;
  for (my $j = 0; $j < $local_y; $j ++)
  {
    $temp = r_v($fp);
    print "Expect $offset: $temp\n";
    $offset += $local_x;
  }

  print "Image data should follow here: reading " . ($local_x * $local_y) . " bytes\n";
  my $image;
  $temp = read($fp,$image,$local_x * $local_y);
  print "Done reading $temp bytes.  Writing.\n";

# write
  my $out_fname = substr($ARGV[1], 0, rindex($ARGV[1], '.'));
  $out_fname = $out_fname . "_" . $i . ".bmp";

  my $fp2;
  open ($fp2, ">$out_fname") or die "Couldn't open $out_fname: $!\n";
  binmode($fp2);
  print $fp2 "BM" . pack('V', (14 + 40 + 1024 + ($local_x * $local_y))) .
                    pack('v', 0) . pack('v', 0) . pack('V', (14 + 40 + 1024));
  print $fp2 pack('V', 40) . pack('V', $local_x) . pack('V', $local_y) .
             pack('v', 1) . pack('v', 8) . pack('V', 0) . pack('V', ($local_x * $local_y)) .
             pack('V', 0) . pack('V', 0) . pack('V', 256) . pack('V', 256);

  for (my $clr = 0; $clr < 256; $clr ++)
  {
    print $fp2 $cmap[$clr][0] . $cmap[$clr][1] . $cmap[$clr][2] . pack('C', 0);
  }
  print $fp2 $image;
  close ($fp2);

  print "All done writing $out_fname!\n";
}

close($fp);
```
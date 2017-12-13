-关于图片的选择和裁剪部分的内容，发现还是有很多需要注意的点。  
/* 场景1：选择一张图片 */  
  private void gotoPickImage() {  
        Intent intent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI);  
        startActivityForResult(intent, REQUEST_PICK_IMAGE);  
  }   
/* 场景2：选择一张图片并裁剪获得一个小图 */  
  private void gotoPickAndCropSmallBitmap() {  
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);  
        intent.setType("image/*");  
        intent.putExtra("crop", "true");  
        intent.putExtra("aspectX", 1);  
        intent.putExtra("aspectY", 1);  
        intent.putExtra("outputX", 300);  
        intent.putExtra("outputY", 300);  
        intent.putExtra("scale", true);  
        intent.putExtra("return-data", true);  
        intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());  
        intent.putExtra("noFaceDetection", true); // no face detection  
        startActivityForResult(intent, REQUEST_CROP_IMAGE_SMALL);  
    }  
/* 场景3：选择一张图片并裁剪获得一个大图 */  
   private void gotoPickAndCropBigBitmap() {  
        imageUri = getTmpUri();  
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT, null);  
        intent.setType("image/*");  
        intent.putExtra("crop", "true");  
        intent.putExtra("aspectX", 1);  
        intent.putExtra("aspectY", 1);  
        intent.putExtra("outputX", 2000);  
        intent.putExtra("outputY", 2000);  
        intent.putExtra("scale", true);  
        intent.putExtra("return-data", false);  
        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);  
        intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());  
        intent.putExtra("noFaceDetection", true); // no face detection  
        startActivityForResult(intent, REQUEST_CROP_IMAGE_BIG);  
    }  
/* 场景4：拍照并裁剪 */  
    private void startImageCapture() {  
        String IMAGE_FILE_LOCATION = Environment.getExternalStorageDirectory() + "/" + "posprint" + "/tmp.jpg";//temp file  
        imageUri = Uri.fromFile(new File(IMAGE_FILE_LOCATION));//The Uri to store the big bitmap
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
        startActivityForResult(intent, REQUEST_CAPTURE_AND_CROP);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (resultCode == Activity.RESULT_OK) {
            Bitmap bitmap = null;
            try {
                switch (requestCode) {
                    case REQUEST_PICK_IMAGE:
                        //选择的图片的Uri
                        imageUri = data.getData(); 
                        bitmap = MediaStore.Images.Media.getBitmap(
                                getContentResolver(), imageUri);
                    case REQUEST_CROP_IMAGE_SMALL:
                        //裁剪后的小图
                        bitmap = data.getParcelableExtra("data");  
                        break;
                    case REQUEST_CROP_IMAGE_BIG:
                        //裁剪后的大图
                        bitmap = MediaStore.Images.Media.getBitmap(getContentResolver(), imageUri);
                        break;
                    case REQUEST_CAPTURE_AND_CROP:
                        //得到拍照后的图片并裁剪
                        cropImageUri(imageUri, REQUEST_CROP_IMAGE_BIG);
                        break;


                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        doSomething(bitmap);
    }

    //获得临时保存图片的Uri，用当前的毫秒值作为文件名
    private Uri getTmpUri() {
        String IMAGE_FILE_DIR = Environment.getExternalStorageDirectory() + "/" + "app_name";
        File dir = new File(IMAGE_FILE_DIR);
        File file = new File(IMAGE_FILE_DIR, Long.toString(System.currentTimeMillis()));
        //非常重要！！！如果文件夹不存在必须先手动创建
        if (!dir.exists()) {
            dir.mkdirs();
        }
        return Uri.fromFile(file);
    }

    //裁剪拍照后得到的图片
    private void cropImageUri(Uri uri, int requestCode) {
        Intent intent = new Intent("com.android.camera.action.CROP");
        intent.setDataAndType(uri, "image/*");
        //intent.putExtra("crop", "true");
        intent.putExtra("aspectX", 1);
        intent.putExtra("aspectY", 1);
        intent.putExtra("outputX", 500);
        intent.putExtra("outputY", 500);
        intent.putExtra("scale", true);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
        intent.putExtra("return-data", false);
        intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
        intent.putExtra("noFaceDetection", true); // no face detection
        intent = Intent.createChooser(intent, "裁剪图片");
        startActivityForResult(intent, requestCode);
    }

# imagick bug

I have some bug (or feature). I want to put multiline text on the center of image.
*Я заметил странный баг (или фичу). Я хочу наложить многострочный текст в центр картинки*

    $my_text_lines = ['sssssssss', 'ddddddddd', 'bbbbbbbbbb'];
    $background_image = new Imagick(dirname(__FILE__).'/test/img.jpg');

    $background_image_1 = $background_image->clone();
    $background_image_2 = $background_image->clone();

It's function for putting text-blocks in two ways for text and putting green lines for check the correct y-position.
*Это функция, которая двумя способами накладывает текстовые блоки и зеленые линии для проверки правильности y-кординат*

    function put_text(Imagick $image, $text_lines) {

        $line_height = 70;

        $text_style = new ImagickDraw();
        $text_style->setFillColor(new ImagickPixel('#ffffff'));
        $text_style->setTextEncoding('utf-8');
        $text_style->setFont(dirname(__FILE__).'/fonts/skolarpe-webfont.ttf');
        $text_style->setFontSize(40);
        $text_style->setFillOpacity(1);

        $start_y_point = ($image->getImageHeight() - count($text_lines)*$line_height)/2;
        for ($i = 0; $i < count($text_lines); $i++) {

            $y_point = $start_y_point + $i*$line_height;

            // default put text on image
            $image->annotateImage(
                $text_style,
                50,
                $y_point,
                0,
                $text_lines[$i]
            );

            // other way to put text on image
            $other_text_draw = new ImagickDraw();
            $other_text_draw->setFillColor(new ImagickPixel('#ff0000'));
            $other_text_draw->setFontSize(30);
            $other_text_draw->setFont(dirname(__FILE__).'/fonts/skolarpe-webfont.ttf');
            $other_text_draw->annotation(150, $y_point, $text_lines[$i]);
            $image->drawImage($other_text_draw);

            // test rectangle
            $test_rectangle = new \ImagickDraw();
            $test_rectangle->setFillColor(new ImagickPixel('#00ff00'));
            $test_rectangle->rectangle(30, $y_point, $image->getImageWidth()-30, $y_point+5);

            $image->drawImage($test_rectangle);

        }

        return $image;
    }
    
On original image everything is fine.
*С оригинальной кортинкой все хорошо*

    $background_image_1 = put_text($background_image_1, $my_text_lines);
    $background_image_1->writeImage(dirname(__FILE__).'/test/test_imagick_1.jpg');
    
In next case I try to crop image and resize it to original size
*В другом варианте я пробую обрезать изображение и изменить его размер до оригинального*

    $background_image_2->cropImage(
        $background_image->getImageWidth()/2,
        $background_image->getImageHeight()/2,
        $background_image->getImageWidth()/4,
        $background_image->getImageHeight()/4);

    $background_image_2->resizeImage(
        $background_image->getImageWidth(), 
        $background_image->getImageHeight(), 
        Imagick::FILTER_LANCZOS, 1, false);
      
    $background_image_3 = $background_image_2->clone();
        
And put text and save it to other file;
*...наложить текст и сохранить его в другом файле*

    $background_image_2 = put_text($background_image_2, $my_text_lines);
    $background_image_2->writeImage(dirname(__FILE__).'/test/test_imagick_2.jpg');

The position of text lines is incorrect (see image below). Why? How put text on correct position?
*Положение строк текста оказывается неправильным (см. картинку ниже). Почему так происходит? Как наложить текст корректно?*

I checked image sizes and resolutions.
*Я проверил и размеры картинки, и разрешение*

I have bad solution: to save the image after crop and resize and re-open it. In this case all fine but it's a very bad solution.
*Есть одно костыльное решение: сохранить картинку после обрезки и изменения размера, а затем переоткрыть ее. В этом случае все работает правильно, но это очень плохое решение.*

    $background_image_3->writeImage(dirname(__FILE__).'/test/tmp.jpg');
    $background_image_3->destroy();
    $background_image_3 = new Imagick(dirname(__FILE__).'/test/tmp.jpg');
    $background_image_3 = put_text($background_image_3, $my_text_lines);
    $background_image_3->writeImage(dirname(__FILE__).'/test/test_imagick_3.jpg');

    <table>
        <tr>
            <td><?php var_dump($background_image_1->getImageResolution()); ?></td>
            <td><?php var_dump($background_image_2->getImageResolution()); ?></td>
            <td><?php var_dump($background_image_3->getImageResolution()); ?></td>
            <td rowspan="3"><?php var_dump(Imagick::getVersion()); ?></td>
        </tr>
        <tr>
            <td><?php echo "{$background_image_1->getImageWidth()}x{$background_image_1->getImageHeight()}"; ?></td>
            <td><?php echo "{$background_image_2->getImageWidth()}x{$background_image_2->getImageHeight()}"; ?></td>
            <td><?php echo "{$background_image_3->getImageWidth()}x{$background_image_2->getImageHeight()}"; ?></td>
        </tr>
        <tr>
            <td><img src="/test/test_imagick_1.jpg" /></td>
            <td><img src="/test/test_imagick_2.jpg" /></td>
            <td><img src="/test/test_imagick_3.jpg" /></td>
        </tr>
    </table>
    
<img src="https://i.stack.imgur.com/E0Zop.png" />

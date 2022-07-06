---
layout: post
title:  "Deploying PyTorch with Flask"
date:   2022-07-05 21:11:12 -0700
categories: pytorch flask ml imagenet
---
I recently came across this [PyTorch tutorial][pytorch-flask-article] and was pleasantly surprised with how easy it was to expose an [DenseNet-121 model][densenet-papers-code] through a REST API. I decided to extend that tutorial and create a web UI as well.

The folder structure eventually looked like this:
```sh
server/
├── templates/
│   ├── index.html
│   └── result.html
├── app.py
└── imagenet_class_index.json
```

### Uploading an Image

The entrypoint to the UI was rendered from the `index.html` template[^sakura]. There, I created an upload form to load and render the image in the same view (see `renderFile`).

By uploading the image, the browser sends a `POST` request to the server with the uploaded image.

{% highlight html %}
{% raw %}
<!-- index.html -->
<head>
  <meta charset="utf-8">
  <title>DenseNet Classifier</title>
  <!-- Use sakura.css to give UI a cleaner look. -->
  <link rel="stylesheet" href="https://unpkg.com/sakura.css/css/sakura.css"
    type="text/css">
  <script>
    // Light JavaScript to load an image.
    var renderFile = function (event) {
      var output = document.getElementById('output');
      output.src = URL.createObjectURL(event.target.files[0]);
      output.onload = function () {
        URL.revokeObjectURL(output.src)  // Free memory.
      }
    };
  </script>
</head>

<body>
  <h1>DenseNet Classifier</h1>
  <label for=file>Upload your image</label>
  <form method=post enctype=multipart/form-data>
    <input type=file accept=image/* onchange=renderFile(event) name=file>
    <input type=submit value=Upload>
  </form>
  <img id="output" />
</body>
{% endraw %}
{% endhighlight %}

### Handling Requests

I handled the `GET` and `POST` requests for loading the initial page and subsequent image upload respectively. I believe there should be an alternative to saving the image to file e.g. getting Flask to render image bytes instead.

{% highlight python %}
@app.route('/', methods=['GET', 'POST'])
def index():
  if request.method == 'POST':
    file = request.files['file']
    filename = secure_filename(file.filename)

    # Save the image to a path in the server's directory.
    # We need this to return the same image to the user.
    # NOTE: This is not the safest thing to do.
    saved_filename = os.path.join(
        app.config['UPLOAD_FOLDER'], filename)
    file.save(saved_filename)

    # Run inference. `class_name` will be used in the 
    # `result.html` template.
    _, class_name = get_prediction(saved_filename)

    # ImageNet classes are underscore-separated. Replace
    # them with spaces for better readability in the UI.
    class_name = class_name.replace('_', ' ')

    # Populate the results template with the class and image.
    return render_template("result.html",
                           class_name=class_name,
                           saved_filename=filename)
  return redirect(request.url)
{% endhighlight %}

### Running Inference

I tweaked `get_prediction` to read an image from a file instead of a image bytes. It worked better for my use case since I was already persisting this image. 

{% highlight python %}
def get_prediction(filename):
    with open(filename, 'rb') as f:
        image_bytes = f.read()
        tensor = transform_image(image_bytes=image_bytes)
    outputs = model.forward(tensor)
    # The tensor y_hat will contain the index of the predicted class id.
    # However, we need a human readable class name. For that we need a class
    # id to name mapping.
    _, y_hat = outputs.max(1)
    predicted_idx = str(y_hat.item())
    return imagenet_class_index[predicted_idx]
{% endhighlight %}

### Rendering Result

To show the results to the client, I created `result.html` to render the image and its class. 
 
{% highlight html %}
{% raw %}
<!-- result.html -->
<body>
  <h1>{{ class_name }}</h1>
  <img src="{{ url_for('static', filename=saved_filename) }}"
    alt="{{ class_name }}">
</body>
{% endraw %}
{% endhighlight %}

## Links

* [Deploying Pytorch in Python via a Rest API With Flask][pytorch-flask-article]
* [DenseNet, Papers With Code][densenet-papers-code]
* [Densely Connected Convolutional Networks, arXiv Paper][densenet-paper]
* [Paper Review: DenseNet–Densely Connected Convolutional Networks][densenet-article]
* [ImageNet][imagenet]
* [sakura: a minimal, classless CSS framework / theme][sakura]

[^sakura]: I used [sakura] to style the UI.
[pytorch-flask-article]: https://pytorch.org/tutorials/intermediate/flask_rest_api_tutorial.html 
[densenet-papers-code]: https://paperswithcode.com/method/densenet
[densenet-paper]: https://arxiv.org/abs/1608.06993
[densenet-article]: https://towardsdatascience.com/paper-review-densenet-densely-connected-convolutional-networks-acf9065dfefb
[imagenet]: https://image-net.org/
[sakura]: https://github.com/oxalorg/sakura/
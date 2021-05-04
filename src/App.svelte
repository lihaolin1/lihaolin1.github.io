<script>
  import _ from "lodash-es";

  import Grid from "./Grid.svelte";

  const ModelTypes = {
    RELATIVE: "relative",
    POSITION_ONLY: "position_only",
    VISION_TRANSFORMER: "vision_transformer"
  };

  $: numberColumns = modelType == ModelTypes.VISION_TRANSFORMER ? 14 : 16;
  let modelType = ModelTypes.VISION_TRANSFORMER;
  let withSoftmax = true;

  const relativeDisplayValues = [
    ["attention (qr+qk)", "attention"],
    ["positional only scores (qr)", "qTr"],
    ["content only scores (qk)", "qTk"]
  ];

  function pad(num, size) {
    var s = "0000000000000000" + num;
    return s.substr(s.length - size);
  }

  const imagePath = {
    [ModelTypes.RELATIVE]: index =>
      "./cifar10-test-images/image_" + index + ".jpg",
    [ModelTypes.POSITION_ONLY]: index =>
      "./cifar10-test-images/image_" + index + ".jpg",
    [ModelTypes.VISION_TRANSFORMER]: index =>
      index == "batch"
        ? "./cifar10-test-images/image_batch.jpg"
        : "./imagenet/image_" + pad(index, 3) + ".png"
  };

  const imageIndices = {
    [ModelTypes.RELATIVE]: ["batch", 2, 3, 19, 56],
    [ModelTypes.POSITION_ONLY]: ["position"],
    [ModelTypes.VISION_TRANSFORMER]: ["batch", 3, 40, 42, 55, 88]
  };

  let selectedContentAttentionType;

  $: selectedColumn = Math.round(numberColumns / 3);
  $: selectedRow = Math.round(numberColumns / 4);

  const onChangeRowColumn = (row, col) => {
    selectedRow = row;
    selectedColumn = col;
  };

  let selectedImages = {};
  selectedImages[ModelTypes.RELATIVE] = "batch";
  selectedImages[ModelTypes.POSITION_ONLY] = "position";
  selectedImages[ModelTypes.VISION_TRANSFORMER] = "batch";

  $: selectedImage = selectedImages[modelType];

  const squareSize = 6;

  const onSelectImage = imageIndex => {
    selectedImages[modelType] = imageIndex;
  };

  const pairsToDict = listOfPairs => {
    return _.zipObject(..._.zip(...listOfPairs));
  };

  $: filename = filenameAt(selectedRow, selectedColumn);

  $: filenameAt = (col, row) => {
    let position;
    if (modelType == ModelTypes.VISION_TRANSFORMER)
      position = col == null ? "cls" : row + "_" + col;
    else position = col == null ? "cls" : col + "_" + row;
    const image_filename =
      "image_" + selectedImage + "/img_" + position + ".png";
    if (modelType == ModelTypes.RELATIVE) {
      let displayValues = selectedContentAttentionType;
      if (displayValues == "attention") {
        displayValues = displayValues + (withSoftmax ? "_probs" : "_scores");
      }
      return "png/learned_qk_qr/" + displayValues + "/" + image_filename;
    } else if (modelType == ModelTypes.POSITION_ONLY)
      return (
        "png/learned/attention" +
        (withSoftmax ? "_probs" : "_scores") +
        "/" +
        image_filename
      );
    else if (modelType == ModelTypes.VISION_TRANSFORMER)
      return "png/vit/" + image_filename;
  };
</script>

<style>
  .sampleImage {
    margin-right: 12px;
    margin-bottom: 8px;
    position: relative;
    float: left;
    /* border: 1px solid black; */
  }

  .imageSelected {
    box-shadow: 3px 3px 3px #69a9d2bb;
    outline: none;
  }

  .selected {
    box-shadow: 0 0 5px #2980b9;
    /* border: 3px solid #2980b9; */
    outline: none;
  }

  /* move the grid on the image it lies over */
  .grid {
    top: 0;
    left: 0;
    position: absolute;
  }

  select {
    display: inline;
  }

  div.clearFloat {
    clear: both;
  }

  .attentionMaps {
    width: 100%;
  }

  .preload img {
    opacity: 0;
    position: absolute;
    width: 0;
    height: 0;
  }

  .modelTypeSelectBox {
    border: 1px solid grey;
    /* outline: 2px solid white; */
    padding: 0.5em 0.8em;
    margin-right: 1em;
    margin-bottom: 0.5em;
    border-radius: 0.5em;
    float: left;
    user-select: none;
  }

  .modelTypeSelectBox:last-child {
    margin-right: 0;
  }

  .modelTypeSelectBox.selected {
    box-shadow: 0 0 5px #2980b9;
  }

  .modelTypeSelectBox h4 {
    margin-top: 0;
    margin-bottom: 0.3em;
    font-variant: small-caps;
    /* font-weight: normal; */
  }

  .imagesContainer {
    margin-left: 3%;
  }
</style>

<main>
  <h1>Visualization of Self-Attention Maps in Vision</h1>

  <p>
    This interactive webpage illustrates the findings of our paper
    <a href="https://openreview.net/pdf?id=HJlnC1rKPB">
      On the Relationship between Self-Attention and Convolutional Layers
    </a>
    published at ICLR 2020. We prove that a Self-Attention layer can express any
    convolution (under basic conditions met in practice) by attending on (groups
    of) pixels at fixed shift of the query pixel. This expressivity result is
    somehow matched in practice for some heads that ignore the content and
    compute a peak attention probability at a fixed shift (up to a symmetry).
  </p>

  <p>
    This page displays interactive attention maps computed by a 6-layer
    self-attention model trained to classify CIFAR-10 images. You can consult
    our
    <a href="http://jbcordonnier.com/posts/attention-cnn/">blog post</a>
    for a gentle introduction to our paper. The code is available on
    <a href="https://github.com/epfml/attention-cnn">Github</a>
    , the experimental setting is detailed in the paper.
  </p>

  <p>
    <b>Edit 4/12/2020:</b>
    We added the visualization of
    <a href="https://arxiv.org/abs/2010.11929">Vision Transformer.</a>
    We used the implementation from
    <a href="https://github.com/rwightman/pytorch-image-models">timm</a>
    and the weights from the
    <a href="https://github.com/google-research/vision_transformer">
      original repository.
    </a>
    ViT-Base/16 is a larger model trained on ImageNet rather than CIFAR-10
    without any image specific architecture choice in the positional encoding.
  </p>

  <h4>Select attention type:</h4>
  <div class="modelTypeSelect">
    <div
      class="modelTypeSelectBox"
      style="width:26%"
      on:click={() => {
        modelType = ModelTypes.RELATIVE;
      }}
      class:selected={modelType == ModelTypes.RELATIVE}>
      <h4>Relative Self-Attention</h4>
      Use 2D relative positional encoding and image content to compute the
      attention.
    </div>
    <div
      class="modelTypeSelectBox"
      style="width:30%"
      on:click={() => {
        modelType = ModelTypes.POSITION_ONLY;
      }}
      class:selected={modelType == ModelTypes.POSITION_ONLY}>
      <h4>Position-only Self-Attention</h4>
      Discard the pixel values and compute the attention scores only on relative
      positions.
    </div>
    <div
      class="modelTypeSelectBox"
      style="width:29%"
      on:click={() => {
        modelType = ModelTypes.VISION_TRANSFORMER;
      }}
      class:selected={modelType == ModelTypes.VISION_TRANSFORMER}>
      <h4>Vision Transformer</h4>
      Use absolute 1D positional encoding and CLS token for classification.
      ViT-Base/16.
    </div>
  </div>

  <div class="clearFloat" />

  {#if modelType == ModelTypes.RELATIVE}
    <br />
    <b>Display values:</b>
    <select bind:value={selectedContentAttentionType}>
      {#each relativeDisplayValues as [display, value]}
        <option type="radio" {value}>{display}</option>
      {/each}
    </select>
    {#if selectedContentAttentionType == 'attention'}
      <b>with softmax</b>
      <input type="checkbox" bind:checked={withSoftmax} />
    {:else}
      <b>without softmax</b>
    {/if}
  {:else if modelType == ModelTypes.POSITION_ONLY}
    <br />
    <b>Display positional attention</b>
    <input type="checkbox" bind:checked={withSoftmax} />
    <b>with softmax</b>
  {/if}

  <h4>Select image and query pixel:</h4>

  <div class="imagesContainer">
    {#each imageIndices[modelType] as index}
      <div
        class="sampleImage"
        style={'height: ' + numberColumns * squareSize + 'px;' + 'padding-left:' + (modelType == ModelTypes.VISION_TRANSFORMER ? squareSize : 0) + 'px'}>
        <img
          src={imagePath[modelType](index)}
          alt=""
          class:imageSelected={index === selectedImage}
          width={numberColumns * squareSize}
          on:click={() => onSelectImage(index)} />
        {#if selectedImage == index}
          <div class="grid">
            <Grid
              displayCLSToken={modelType == ModelTypes.VISION_TRANSFORMER}
              columns={numberColumns}
              rows={numberColumns}
              {selectedColumn}
              {selectedRow}
              {onChangeRowColumn}
              size={squareSize}
              class="grid" />
          </div>
        {/if}
      </div>
    {/each}
  </div>

  <div class="clearFloat" />

  <h4>Visualize attention per layer and head</h4>

  <img src={filename} alt="" class="attentionMaps" />

  <div class="preload">
    {#each _.range(numberColumns) as col}
      {#each _.range(numberColumns) as row}
        <img src={filenameAt(col, row)} alt="preload" />
      {/each}
    {/each}
  </div>

</main>

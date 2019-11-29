<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/PDF_file_icon.svg/833px-PDF_file_icon.svg.png" height="128">

# compare-pdf

Standalone node module that compares pdfs

## Setup

Install the following system dependencies

-   [GraphicsMagick](http://www.graphicsmagick.org/README.html)
-   [ImageMagick](https://imagemagick.org/script/download.php)
-   [GhostScript](https://www.ghostscript.com/download.html)

Install npm module

```
npm install compare-pdf
```

## Default Configuration

Below is the default configuration showing the paths where the pdfs should be placed. By default, they are in the root folder of your project inside the folder data.

```
{
    paths: {
        actualPdfRootFolder: process.cwd() + "/data/actualPdfs",
        baselinePdfRootFolder: process.cwd() + "/data/baselinePdfs",
        actualPngRootFolder: process.cwd() + "/data/actualPngs",
        baselinePngRootFolder: process.cwd() + "/data/baselinePngs",
        diffPngRootFolder: process.cwd() + "/data/diffPngs"
    },
    settings: {
        density: 100,
        quality: 70,
        tolerance: 0,
        threshold: 0.05
    }
}
```

## Compare Pdfs By Image

### Basic Usage

By default, pdfs are compared using the comparison type as "byImage"

```
it("Should be able to verify same PDFs", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("same.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});

it("Should be able to verify different PDFs", async () => {
    const ComparePdf = new comparePdf();
    let comparisonResults = await ComparePdf.actualPdfFile("notSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare("byImage");
    expect(comparisonResults.status).to.equal("failed");
    expect(comparisonResults.message).to.equal("notSame.pdf is not the same as baseline.pdf.");
    expect(comparisonResults.details).to.not.be.null;
});
```

### Using Masks

You can mask areas of the images that has dynamic values (ie. Dates, or Ids) before the comparison. Just use the addMask method and indicate the pageIndex (starts at 0) and the coordinates.

```
it("Should be able to verify same PDFs with Masks", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("maskedSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .addMask(1, { x0: 35, y0: 70, x1: 145, y1: 95 })
        .addMask(1, { x0: 185, y0: 70, x1: 285, y1: 95 })
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

You can also indicate the page masks in bulk by passing an array of it in the addMasks method

```
it("Should be able to verify different PDFs with Masks", async () => {
    const ComparePdf = new comparePdf();
    let masks = [
        { pageIndex: 1, coordinates: { x0: 35, y0: 70, x1: 145, y1: 95 } },
        { pageIndex: 1, coordinates: { x0: 185, y0: 70, x1: 285, y1: 95 } }
    ];
    let comparisonResults = await ComparePdf.actualPdfFile("maskedNotSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .addMasks(masks)
        .compare();
    expect(comparisonResults.status).to.equal("failed");
    expect(comparisonResults.message).to.equal("maskedNotSame.pdf is not the same as baseline.pdf.");
    expect(comparisonResults.details).to.not.be.null;
});
```

### Cropping Pages

If you need to compare only a certain area of the pdf, you can do so by utilising the cropPage method and passing the pageIndex (starts at 0), the width and height along with the x and y coordinates.

```
it("Should be able to verify same PDFs with Croppings", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("same.pdf")
        .baselinePdfFile("baseline.pdf")
        .cropPage(1, { width: 530, height: 210, x: 0, y: 415 })
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

Similar to masks, you can also pass all cropping in bulk into the cropPages method

```
it("Should be able to verify same PDFs with Croppings", async () => {
    let croppings = [
        { pageIndex: 0, coordinates: { width: 210, height: 180, x: 615, y: 770 } },
        { pageIndex: 1, coordinates: { width: 530, height: 210, x: 0, y: 415 } }
    ];

    let comparisonResults = await new comparePdf()
        .actualPdfFile("same.pdf")
        .baselinePdfFile("baseline.pdf")
        .cropPages(croppings)
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

## Compare Pdfs By Base64

### Basic Usage

By passing "byBase64" as the comparison type parameter in the compare method, the pdfs will be compared whether the actual and baseline's converted file in base64 format are the same.

```
it("Should be able to verify same PDFs", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("same.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare("byBase64");
    expect(comparisonResults.status).to.equal("passed");
});

it("Should be able to verify different PDFs", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("notSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare("byBase64");
    expect(comparisonResults.status).to.equal("failed");
    expect(comparisonResults.message).to.equal("notSame.pdf is not the same as baseline.pdf.");
});
```

## Other Capabilities

### Overriding Default Configuration

Users can override the default configuration by passing their custom config when initialising the class

```
it("Should be able to override default configs", async () => {
    let config = {
        paths: {
            actualPdfRootFolder: process.cwd() + "/data/newActualPdfs",
            baselinePdfRootFolder: process.cwd() + "/data/baselinePdfs",
            actualPngRootFolder: process.cwd() + "/data/actualPngs",
            baselinePngRootFolder: process.cwd() + "/data/baselinePngs",
            diffPngRootFolder: process.cwd() + "/data/diffPngs"
        },
        settings: {
            density: 100,
            quality: 70,
            tolerance: 0,
            threshold: 0.05
    };
    let comparisonResults = await new comparePdf(config)
        .actualPdfFile("newSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});

it("Should be able to override specific config property", async () => {
    const ComparePdf = new comparePdf();
    ComparePdf.config.paths.actualPdfRootFolder = process.cwd() + "/data/newActualPdfs";
    let comparisonResults = await ComparePdf.actualPdfFile("newSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

### Pdf File Paths

Users can pass just the filename with or without extension as long as the pdfs are inside the default or custom configured actual and baseline paths

```
it("Should be able to pass just the name of the pdf with extension", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("same.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});

it("Should be able to pass just the name of the pdf without extension", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("same")
        .baselinePdfFile("baseline")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

Users can also pass a relative path of the pdf files as parameters

```
it("Should be able to verify same PDFs using relative paths", async () => {
    let comparisonResults = await new comparePdf()
        .actualPdfFile("../data/actualPdfs/same.pdf")
        .baselinePdfFile("../data/baselinePdfs/baseline.pdf")
        .compare();
    expect(comparisonResults.status).to.equal("passed");
});
```

### Tips and Tricks

To speed up your test executions, you can utilise the comparison type "byBase64" first and only when it fails you comapre it "byImage". This provides the best of both worlds where you get the speed of execution and when there is a difference, you can check the image diff.

```
it("Should be able to verify PDFs byBase64 and when it fails then byImage", async () => {
    let comparisonResultsByBase64 = await new comparePdf()
        .actualPdfFile("notSame.pdf")
        .baselinePdfFile("baseline.pdf")
        .compare("byBase64");
    expect(comparisonResultsByBase64.status).to.equal("failed");
    expect(comparisonResultsByBase64.message).to.equal(
        "notSame.pdf is not the same as baseline.pdf compared by their base64 values."
    );

    if (comparisonResultsByBase64.status === "failed") {
        let comparisonResultsByImage = await new comparePdf()
            .actualPdfFile("notSame.pdf")
            .baselinePdfFile("baseline.pdf")
            .compare("byImage");
        expect(comparisonResultsByImage.status).to.equal("failed");
        expect(comparisonResultsByImage.message).to.equal(
            "notSame.pdf is not the same as baseline.pdf compared by their images."
        );
        expect(comparisonResultsByImage.details).to.not.be.null;
    }
});
```

macOS users encountering "dyld: Library not loaded" error? Then follow the answer from this [stackoverflow post](https://stackoverflow.com/questions/55754551/how-to-install-imagemagick-portably-on-macos-when-i-cant-set-dyld-library-path) to set the correct path to \*.dylib.

## Example Projects

-   [Using Mocha + Chai](/examples/mocha)
-   [Using Cypress](/examples/cypress)
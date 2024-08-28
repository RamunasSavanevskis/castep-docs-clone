# C. Phonon Dispersion and DOS

In this tutorial, we will examine the phonon DOS and dispersion curves of rocksalt NaH, and examine how the results may be improved (and run quicker) using continuations.

We will use the `cell` file

*nah.cell*
```
%BLOCK LATTICE_ABC
3.393405 3.393405 3.393405
60 60 60
%ENDBLOCK LATTICE_ABC

%BLOCK POSITIONS_FRAC
Na 0 0 0
H 0.5 0.5 0.5
%ENDBLOCK POSITIONS_FRAC

SYMMETRY_GENERATE

%BLOCK SPECIES_POT
NCP
%ENDBLOCK SPECIES_POT

PHONON_KPOINT_MP_GRID 4 4 4
phonon_kpoint_mp_offset 0.125 0.125 0.125
KPOINTS_MP_GRID 4 4 4
```

and the `param` file

*nah.param*
```
task : PHONON
basis_precision : fine
xc_functional : LDA
opt_strategy : SPEED
fix_occupancy : TRUE
elec_method : DM
phonon_sum_rule : TRUE
phonon_fine_method : INTERPOLATE
#continuation : nah.check
```

The flowchart below outlines the steps and results for getting and continuing the DOS and bandstructure results. Any `.pl` boxes are what to put in the command line, and the other boxes are information to put into the `cell` file. For continuations, do not remove the "non-fine" information - so the lines

```
PHONON_KPOINT_MP_GRID 4 4 4
phonon_kpoint_mp_offset 0.125 0.125 0.125
```

should remains, with the "fine" information added in as specified in the flowchart below. Make sure to uncomment `#continuation : nah.check` when performing a continuation - `phonon_fine_method : INTERPOLATE` simply doesn't do anything if you're not calculating any fine information - it only plays a role during the continuations in this example.  

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image Viewer Modal</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }

        .modal {
            display: flex;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.8);
            justify-content: center;
            align-items: center;
        }

        .modal-content-zoomed-in {
            background: #000;
            padding: 0;
            border: 1px solid #888;
            width: calc(85vh * 0.6988);
            height: 85vh;
            position: relative;
            overflow: hidden;
        }

        .image-container {
            width: 100%;
            height: 100%;
            overflow: auto;
            position: relative;
            background: #000;
            cursor: grab;
        }

        .image-container:active {
            cursor: grabbing;
        }

        .viewport {
            position: absolute;
            top: 0;
            left: 0;
            background: url('../flowchart.png') no-repeat;
            background-size: cover;
            background-position: center;
        }

        .close {
            color: #aaa;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
            position: absolute;
            top: 10px;
            right: 10px;
        }

        .close:hover,
        .close:focus {
            color: #fff;
            text-decoration: none;
        }

        .thumbnail {
            cursor: pointer;
            width: 300px;
            height: auto;
            margin: 20px;
        }
    </style>
</head>
<div id="image-viewer-container">
    <img src="../flowchart.png" alt="Thumbnail" class="thumbnail" id="thumbnail">
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
    const container = document.getElementById('image-viewer-container');
    const imageWidth = 2958;
    const imageHeight = 4233;

    ZOOM_INCREMENT = 0.02; // Zoom increment/Decrement value for each scroll

    let zoomLevel; // Variable to store the current zoom level
    let mouseX, mouseY; // Variables to store mouse position relative to the image container
    let imageContainer; // Variable to store the image container element
    let viewport; // Variable to store the viewport element
    let isPanning = false; // Variable to track if panning is active
    let startX, startY, scrollLeft, scrollTop; // Variables for panning

    function calculateInitialZoomLevel(modalWidth, modalHeight) {
        const widthRatio = modalWidth / imageWidth;
        const heightRatio = modalHeight / imageHeight;
        return Math.min(widthRatio, heightRatio);
    }

    function updateZoom(change=0, ZOOM_INCREMENT) {
        const zoomedWidth = imageWidth * zoomLevel;
        const zoomedHeight = imageHeight * zoomLevel;

        const viewportWidth = imageContainer.clientWidth;
        const viewportHeight = imageContainer.clientHeight;

        const effectiveWidth = Math.min(imageWidth, viewportWidth / zoomLevel);
        const effectiveHeight = Math.min(imageHeight, viewportHeight / zoomLevel);

        const mouseXPercentImage1 = (mouseX + imageContainer.scrollLeft) / (imageWidth * (zoomLevel-change*ZOOM_INCREMENT));
        const mouseYPercentImage1 = (mouseY + imageContainer.scrollTop) / (imageHeight * (zoomLevel-change*ZOOM_INCREMENT));
        const mouseXPercentModal = mouseX / viewportWidth;
        const mouseYPercentModal = mouseY / viewportHeight;

        const pixelX = mouseXPercentImage1 * imageWidth;
        const pixelY = mouseYPercentImage1 * imageHeight;

        const newPixelX = mouseXPercentModal * effectiveWidth;
        const newPixelY = mouseYPercentModal * effectiveHeight;

        const pixelShiftX = pixelX - newPixelX ;
        const pixelShiftY = pixelY - newPixelY;

        const adjustedShiftX = pixelShiftX * (viewportWidth/effectiveWidth);
        const adjustedShiftY = pixelShiftY * (viewportHeight/effectiveHeight);

        viewport.style.width = `${zoomedWidth}px`;
        viewport.style.height = `${zoomedHeight}px`;

        imageContainer.scrollLeft = adjustedShiftX;
        imageContainer.scrollTop = adjustedShiftY;
    }

    function openZoomedInModal() {
        const modal = document.createElement("div");
        modal.className = "modal";

        const modalContent = document.createElement("div");
        modalContent.className = "modal-content-zoomed-in";
        modalContent.style.width = 'calc(85vh * 0.6988)'; // Updated width
        modalContent.style.height = '85vh'; // Updated height

        const closeBtn = document.createElement("span");
        closeBtn.className = "close";
        closeBtn.innerHTML = "&times;";
        closeBtn.onclick = () => container.removeChild(modal); // Modified to use container

        imageContainer = document.createElement("div");
        imageContainer.className = "image-container";

        viewport = document.createElement("div");
        viewport.className = "viewport";
        viewport.style.backgroundImage = "url('../flowchart.png')";

        imageContainer.appendChild(viewport);
        modalContent.appendChild(closeBtn);
        modalContent.appendChild(imageContainer);
        modal.appendChild(modalContent);
        container.appendChild(modal); // Modified to use container

        const modalWidth = modalContent.clientWidth;
        const modalHeight = modalContent.clientHeight;
        zoomLevel = calculateInitialZoomLevel(modalWidth, modalHeight);

        MIN_ZOOM = zoomLevel;
        updateZoom();

        imageContainer.addEventListener('wheel', function(event) {
            event.preventDefault();

            const rect = imageContainer.getBoundingClientRect();
            mouseX = event.clientX - rect.left;
            mouseY = event.clientY - rect.top;

            if (event.deltaY < 0) {
                ZOOM_INCREMENT = 0.02;
                zoomLevel += ZOOM_INCREMENT; // Zoom in
                change = 1;
            } else {
                ZOOM_INCREMENT = 0.04;
                zoomLevel = Math.max(MIN_ZOOM, zoomLevel - ZOOM_INCREMENT); // Zoom out

                change = -1;
                if (zoomLevel === MIN_ZOOM) {
                    change = 0;  
                }
            }

            updateZoom(change, ZOOM_INCREMENT);
        });

        // Panning functionality
        imageContainer.addEventListener('mousedown', function(event) {
            isPanning = true;
            startX = event.clientX;
            startY = event.clientY;
            scrollLeft = imageContainer.scrollLeft;
            scrollTop = imageContainer.scrollTop;
        });

        imageContainer.addEventListener('mousemove', function(event) {
            if (!isPanning) return;
            const x = event.clientX - startX;
            const y = event.clientY - startY;
            imageContainer.scrollLeft = scrollLeft - x;
            imageContainer.scrollTop = scrollTop - y;
        });

        imageContainer.addEventListener('mouseup', function() {
            isPanning = false;
        });

        imageContainer.addEventListener('mouseleave', function() {
            isPanning = false;
        });

        // Close the modal if the user clicks outside of the modal content
        window.addEventListener('click', function(event) {
            if (event.target === modal) {
                container.removeChild(modal);
            }
        });
    }

    document.getElementById("thumbnail").addEventListener('click', openZoomedInModal);
});
</script>

</html>

It is generally best to start off with a small grid calculation, as we did here with the line `PHONON_KPOINT_MP_GRID 4 4 4`, and then use interpolation to either get a finer grid or to find the results on certain paths for a bandstructure. There is an option to start with `PHONON_KPOINT_MP_PATH`, but that is significantly slower - starting with a grid is generally the better approach.

## C.II Phonon DOS using interpolation

The task here is to use the NaH example to compute and display not a
dispersion curve but a density of states. This will exploit CASTEPs
interpolation functionality, and you will be able to compute a good DOS
without the need the repeat the expensive electronic structure
calculation. To do this you will need to set up a calculation neatly
identical to your previous one but with two differences.

1.  The calculation should be set up as a continuation. If your previous
    run wrote a .check file named “NaH-disp.check” for example, then the
    param file should contain the line

     * continuation : NaH-disp.check*

2.  Instead of a %block phonon_fine_kpoint_list in the .cell file, you
    can specify a grid

    *phonon_fine_kpoint_mp_grid 16 16 16*

You can run CASTEP on just 1-4 processors for this

You can try this several times with different fine q-point grids.

This will produce a .castep and .phonon file as before. You may analyse
the .phonon file and generate a DOS using the “dos.pl” script

> *dos.pl -xg NaH-dos.phonon*

(again, an X server running on the PC will be needed for grace to
display) .

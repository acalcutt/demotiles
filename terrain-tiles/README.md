

# Terrain Tiles

Contains terrain tiles in the N047E011 region. These tiles are made to be used with our terrain demo page, which centers around Innsbruck Austria [11.40416, 47.26475]

## How to use the demo Terrain Tiles

1.) Include the terrain tiles in your maps list of sources.

    terrain: {
    	type: 'raster-dem',
    	url: 'https://demotiles.maplibre.org/terrain-tiles/tiles.json',
    	tileSize: 256
    },

2.) Use the terrain tile source for 3d terrain in you style.

    terrain: {
    	source:  'terrain',
    	exaggeration:  1
    }

3.) Use the terrain tile source for hillshade.

    {
    	id:  'hills',
    	type:  'hillshade',
    	source:  'terrain',
    	layout: {'visibility':  'visible'},
    	paint: {'hillshade-exaggeration':  0.33}
    }
  
4.) With all of these combined, the style should look something like this.
 

    style: {
    	version:  8,
    	sources: {
    		osm: {
				type:  'raster',
				tiles: ['https://a.tile.openstreetmap.org/{z}/{x}/{y}.png'],
				tileSize:  256,
				attribution:  '&copy; OpenStreetMap Contributors',
				maxzoom:  19
			},
    		terrain: {
				type:  'raster-dem',
				url:  'https://demotiles.maplibre.org/terrain-tiles/tiles.json',
				tileSize:  256
			},
    	},
    	layers: [
    		{
    			id:  'osm',
    			type:  'raster',
    			source:  'osm'
    		},
    		{
    			id:  'hills',
				type:  'hillshade',
				source:  'terrain',
				layout: {'visibility':  'visible'},
				paint: {'hillshade-exaggeration':  0.33}
			}
		],
		terrain: {
			source:  'terrain',
			exaggeration:  1
		}
    }

5.) With all this in place, view it by going to Innsbruck, Austria [11.40416, 47.26475] on your map. These tiles only cover a 1 degree x 1 degree area, so they are only viewable in the N047E011 region.

## Terrain Tile Generation


1.) This terrain was made from JAXA AW3D30 DSM files. To get these yourself, you must register for a free account at https://www.eorc.jaxa.jp/ALOS/en/dataset/aw3d30/aw3d30_e.htm .

2.) Once registered, go to the JAXA [download page](https://www.eorc.jaxa.jp/ALOS/en/aw3d30/data/index.htm) and get the ZIP files which match the region you would like to map.

3.) From each zip file you downloaded, extract the file that ends in "_DSM.tif" and place it into a folder.

4.) Run the following bash script. Make sure the INPUT_DIR directory is the folder where you placed the "_DSM.tif" files. This requires proj, gdal, and rio-rgbify to be installed.

    #!/bin/bash
    
    INPUT_DIR=./input
    OUTPUT_DIR=./rio
    vrtfile=${OUTPUT_DIR}/jaxa_terrainrgb_0-12.vrt
    vrtfile2=${OUTPUT_DIR}/jaxa_terrainrgb_0-12_warp.vrt
    mbtiles=${OUTPUT_DIR}/jaxa_terrainrgb_0-12.mbtiles

    [ -d "$OUTPUT_DIR" ] || mkdir -p $OUTPUT_DIR || { echo "error: $OUTPUT_DIR " 1>&2; exit 1; }
    
    #rm rio/*
    gdalbuildvrt -overwrite -srcnodata -9999 -vrtnodata -9999 ${vrtfile} ${INPUT_DIR}/*_DSM.tif
    gdalwarp -r cubicspline -t_srs EPSG:3857 -dstnodata 0 -co COMPRESS=DEFLATE ${vrtfile} ${vrtfile2}
    rio rgbify -b -10000 -i 0.1 --min-z 0 --max-z 12 -j 24 --format png ${vrtfile2} ${mbtiles}
    
    #CREATE UNIQUE INDEX tile_index on tiles (zoom_level, tile_column, tile_row);
    sqlite3 ${mbtiles} 'CREATE UNIQUE INDEX tile_index on tiles (zoom_level, tile_column, tile_row);'

5.) You now should have a TerrainRGB mbtiles file in the OUTPUT_DIR of the script. 

6.) Use a sqlite editor to add in attribution, name, description.  The attribution is important to comply with the JAXA [use policy](https://earth.jaxa.jp/policy/en.html) 

7.) To make the mbtiles into flat files (optional), use a tool like mbutil to extract them.

    mb-util jaxa_terrainrgb_0-12.mbtiles terrainrgb

The `tiles.json` file was generated by running `tileserver-gl` with the `.mbtiles` file as an input.

## Data Source
Japan Aerospace Exploration Agency (JAXA)
ALOS Global Digital Surface Model "ALOS World 3D - 30m (AW3D30)"
https://earth.jaxa.jp/en/data/policy/

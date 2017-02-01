# Test Android Renderer (part 3)

tester on my box is under C:\Android\Rendering and uses the Renderer library which is under SVN ..\Rendering\Android.

# AVD setup
It runs much faster with AVD's which are set to Intel x86 CPU. But to run with these you need HAXM installed and hardware virtualization must be set on your computer. Use Target: Google API's x86 - API level 19. Set SD Card size to 20 MiB to be able to pull the temp.kml from storage at runtime. Select Use Host GPU.

# Running the demo
The application opens by displaying the Modifiers menu.Type the 15 character Mil-Std-2525 symbol id or acronym (e.g. flot) in the line type entry at the top of the form. Type the corresponding attributes and modifiers in the lines below. Line color and fill color are hex strings in BGR or ABGR format, e.g. ff0000, or ffff0000. Click the menu button and select Attributes. Enter the appropriate comma delimited strings for AM_DISTANCE in meters (e.g.6000,4000,8000), AN_AZIMUTH in degrees (e.g. 45,115), and X_ALTITUDE_DEPTH in feet (e.g. 24,77). Enter the display extents: left longitude, right longitude, upper latitude, lower latitude, e,g, 50,51,9,8. Enter the Mil-Std-2525 revision (default is rev C). Click the menu button again. Click Draw option option and left mouse down 4 or fewer points on the display. Shapes requiring a fixed number of anchor points (Autoshapes, e.g. Breach) will display after 4 points or fewer. Click the menu button again, then select Modifiers to begin again with a new symbol. The values entered on the previous iteration will persist on the form. 
 
The tester also tests SECWebRenderer for KML generation for 3D display on the same iteration, so it's calling both functions. To access the KML file at runtime get a command prompt and change directory to android\platform-tools under wherever you did the Android install (mine is under c:\program files (x86)\android-sdk). The internal file generated by the tester will be storage/sdcard/KML/temp.kml. At the prompt type adb pull to wherever you want to some folder, e.g. adb pull storage/sdcard/temp.kml c:\temp. Open the file in a KML viewer, e.g. Google Earth, to view.

# Developer Notes.

Most symbols can be rendered as shown in sample code from com.example.utility:
DoubleClickGE for rendering with the Canvas.
DoubleClickSECRenderer for KML generation. All of the symbols will render. However only the 3d airspaces (route, track, polygon, polyarc, radarc, orbit, cylinder) and the Mil-Std-2525 c2G/Aviation/Lines (UAV, LLTR, SAAFR, AC, MRR) exhibit the altitudes and hence a 3d appearance when the KML is displayed.

2D (Draw with Canvas).
source: com.example.utility.DoubleClickGE.
clipping area is set to the visible region (clipArea). 
rev is the Mil-Std-2525 version or USAS. The default is rev C.

Buiding the symbol object:	MilStdSymbol mss = CreateMSS(symbolCode, "0", pts2);
Setting the symbol properties, e.g. line color, fill color, modifiers, can be seen in the CreateMSS function. Most of the guidance for setting symbol attributes and modifiers can be determined from Mil-Std-2525, including setting the modifier distance (AM), azimuth (AN) and altitude (X) arrays. Examples of how to set these arrays are found in CreateMSS.

Rendering the symbol:	
The renderable shape objects are returned in the MilStdsymbol with the following call:
	clsRenderer.renderWithPolylines(mss, converter, clipArea, context); 
Drawing the shape objects on the Canvas is done with this call: 
        drawShapeInfosGE(g2d, mss.getSymbolShapes());
The function extracts the the symbol shapes returned by the renderer into an Android Path and drawn to the Android canvas using canvas.drawPath.
Rendering rotatable text is handled with the following call which adds marker objects to the map: 
        drawShapeInfosText(g2d, mss.getModifierShapes());
Clients setting useDashArray in the renderer may be able to speed performance for symbols with dashed lines by calculating the dashed lines instead of allowing the renderer to do it because the renderer creates a new array object ofr each of the dash segments (2 points). See comments in the tester. 

3D (KML Generation).
Source com.example.utility.DoubleClickSECRenderer.
For Mil-Std-2525 symbols the attributes and modifier arrays are set using SparseArrays as shown in the code immediately preceding the line:
            String strRender = sec.RenderSymbol("id", "name", "description", defaultText, controlPtsStr, altitudeMode, scale, rectStr, modifiers, attributes, 0, rev);
As for the 2D analysis above, the modifiers and attributes requrements can be deduced from Mil-Std-2525 or the USAS.
For Airspace symbols the attributes are set using JSON strings as shown in the code immediately preceding the line:
            strCake = sec.Render3dSymbol("name", "id", defaultText, "", "ff0000ff", "", controlPtsStr, acAttributes);
The requirements for setting the Airspace attributes is as follows:
The symbol id is created from the generic name and dashes to length 15, e.g. symbol id for route is "ROUTE----------". The array requirements are as follows:
Route: radius1 AM[0], minalt X[0], maxalt X[1],
Cylinder: radius1 AM[0], minalt X[0], maxalt X[1]
Orbit: radius1 AM[0], minalt X[0], maxalt X[1]
Polygon: minalt X[0], maxalt X[1]
Radarc: radius1 AM[0], radius2 AM[1], leftAzimuth AN[0], rightAzimuth AN[1], minalt X[0], maxalt X[1]
Polyarc:  radius1 AM[0], leftAzimuth AN[0], rightAzimuth AN[1], minalt X[0], maxalt X[1]
Track: for each segment n: radius1 AM[2*n], radius2 AM[2*n+1], minalt X[2*n], maxalt X[2*n+1]
Radii are in meters, azimuths are in degrees, altitudes are in feet. At runtime the resultant KML string is parsed to a KML file:
                parseKML.parseLLTR(strCake, coordStrings);
The KML file can be pulled to the user's hard drive per instructions above using adb pull.
Note: for airspaces there is also a 2D rendering (no altitudes used) by the following lines:
                drawSECRendererCoords3d(g2d, coordStrings, converter)


Attributes, use the following array sizes:
Ciruclar range fans: AM[n] n is the number of radii
Sector range fans: AM[n], AN[left,right,...n-1 times] X[n], n is number of sectors
Rectangular Target: AM[2], AN[1]
Cirular Target: AM[1]
FSA rectangular (and others): AM[1]
FSA circular (and others): AM[1]
ACA: X[2]

Airspaces
Track: AM[2(n-1)], X[2(n-1)], n is the number of points 
Route: X[2]
Radarc: AM[2], AN[2], X[2]
Polygon: X[2]
Polyarc: AM[1], AN[2], X[2]
Orbit: AM[1], X[2]
Cylinder: AM[1], X[2]

Mil-Std-2525 3D graphics (Rev C)
Same as Polygon for these:
ACA Irregular
Kill Box Irregular
ROZ
SHORADEZ
HIDACZ
MEZ
LOMEZ
HIMEZ

Same as Cylinder for these:
ACA Circular
Kill box Circular

Same as Track for these:
ACA Rectangular
Kill Box Rectangular
AC
MRR
SAAFR
LLTR
UA (Unmanned Aircraft)







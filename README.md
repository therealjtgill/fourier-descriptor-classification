# fourier-descriptor-classification
README

Jonathan Gill
ECE 532 - Final Project
Dr. Rodriguez
Fall 2016

Instructions:

	Classification Test
		1. Set Matlab's working directory to the location containing these files.
		2. Run 'Descriptors.m' to load data from the training set and see how each descriptor method performs as a classifier on the last 
		   100 training points. The centroid-distance function should have a 74% success rate, the polygonal approximation should have about
		   51% success rate. It can take a few minutes to read and process everything.
		   
	Cross-Validation
		1. Follow the steps from "Classification Test", ignore the output from the steps.
		2. Make the following function call (note that it can take a while to complete):
		
			[avg, unc] = DescriptorCrossVal(distDescriptors, labels, 1, 3, 1, 3)
			
		   The output that you get from this call is a matrix of average training losses for numbers of neighbors and descriptors
		   corresponding to (rows) and (columns) of 'avg', with uncertainties in 'unc'.
		   
		   You can perform the same cross-validation for polygonal descriptors by switching the argument 'distDescriptors' with 
		   'polyDescriptors'.
		   
	Polygon Reconstruction (fun)
		1. Set Matlab's working directory to the location containing these files.
		2. Run "PolygonReconstruction.m" as a script.
	
Miscellaneous Files
--------------------------------------------------------------------------------------------------------------------------------------
ece532.png
startest.png
optdigits_orig_edited.tra

There are several pieces of code included with this package. Below are brief descriptions of each of the core components along with
their inputs and outputs.
--------------------------------------------------------------------------------------------------------------------------------------
Descriptors.m --> script

	Loads handwritten digit data from 'optdigits_orig_edited.tra', assuming that this file is located in the same directory assuming
	Descriptors.m. After loading the data it builds the descriptors for each handwritten digit in the input file, builds a KNN
	classifier for both descriptor methods for all data except the last 100 data points, and tests the last 100 data points against
	the rest of the data.
	
	Args: none.
	
	Returns: none.
	
	However, this script does create several global variables, the most important of which are the shape descriptors and labels:
	
	polyDescriptors
	distDescriptors
	labels
	
	These can be used with "DescriptorCrossVal.m" to determine how well the descriptors work as classifiers.
--------------------------------------------------------------------------------------------------------------------------------------
DescriptorCrossVal.m --> function

	Performs K-Nearest Neighbors classification cross-validation for a set of descriptors obtained using one method (either
	centroid-distance or the polygonal approximation). Training and cross-validation on all supplied descriptors is performed over a
	user-specified range of values for number of neighbors and number of descriptors to use for classification.
	NOTE THAT THIS FUNCTION CAN RUN VERY SLOWLY IF PARAMETER RANGES ARE ABOVE 5.
	
	Args: descriptors - The set of descriptors for multiple objects. Typically this is generated by "Descriptors.m".
		  labels - The labels for each object in the training data; indices of these match up to indices of the "descriptors" input
				   for this function.
		  min/maxDescriptors - The minimum/maximum number of descriptors the user wishes to use to attempt to classify a particular
							   handwritten digit. Note that a KNN classifier is tested and cross-validated for each number of
							   descriptors between the min and max (inclusive).
		  min/maxNeighbors - The minimum/maximum number of neighbors the user wishes to influence the classification of testing data
							 once the KNN model is trained. Note that a KNN classifier is tested and cross-validated for each number
							 of neighborss between the min and max (inclusive).
	
--------------------------------------------------------------------------------------------------------------------------------------
DistanceDescriptors.m --> function

	Creates descriptors based on a given contour's distance function. This function generates the distance function as an intermediate
	step to creating the object's descriptors.
	
	Args: contourPoints - A one-dimensional cell containing tuples of coordinates that, when accessed in sequential order, track the
						  contour of a given object.
	
	Returns: descriptors - A vector of algebraically scaled Fourier coefficients that can be used to train or test the classifier
						   contained in "TestLastHundred.m"
			 distances - A vector representing the centroid-distance function of the contour that was passed to the function.
--------------------------------------------------------------------------------------------------------------------------------------
KNNFourierFeatures.m --> function

	This function should not be run manually. It is a helper function for "DescriptorCrossVal.m".
	
	Args: none.
	
	Returns: none.
--------------------------------------------------------------------------------------------------------------------------------------
Neighborhood8.m --> function

	A helper file for "TraverseOuterContour.m," this function simply returns a 3x3 matrix of pixel values corresponding to a
	neighborhood of some pixel in a bilevel image.
	
	Args: I - a bilevel image file.
		  c - the center of the neighborhood.
		  
	Returns: Neighborhood - A 3x3 matrix of pixel values centered about the function argument 'c'.
--------------------------------------------------------------------------------------------------------------------------------------
PlotFourierSeries.m --> function

	Uses the complex coefficients generated by the PolyDescriptors method to recreate a given contour as a polygonal approximation of
	its original form. Three separate plots are created with 10, 30, and 100 Fourier coefficients
	
	Args: a - The 'positive' frequency coefficients of a complex Fourier series.
		  b - The 'negative' frequency coefficients of a complex Fourier series.
		  label - A name to give the reconstructed contour when plotted.
	
	Returns: none.
--------------------------------------------------------------------------------------------------------------------------------------
PolyDescriptors.m --> function

	Uses a contour that is passed to it to create transformation invariant descriptors for that contour. These descriptors can be used
	to train the KNN model or to get a prediction out of the "TestLastHundred.m" file. The descriptors are generated by turning treating
	the pixels along the discrete contour as corners of a polygon, translating those corners into complex space, and connecting the corners
	with parametrized line segments in the complex plane. Theory supporting this transformation is provided in the paper.
	
	Args: contourPoints - A one-dimensional cell containing tuples of coordinates that, when accessed in sequential order, track the
						  contour of a given object.
	
	Returns: descriptors - A vector of algebraically scaled Fourier coefficients that can be used to train or test the classifier
						   contained in "TestLastHundred.m"
			 fourierCoefficients1 - The "positive" frequency coefficients of the Fourier series generated from polygon-izing the contour.
			 fourierCoefficients2 - The "negative" frequency coefficients of the Fourier series generated from polygon-izing the contour.
--------------------------------------------------------------------------------------------------------------------------------------
PolygonReconstruction.m --> script

	A fun script that reconstructs two polygon-approximated contours, created from bilevel images, with an increasing number of Fourier
	series coefficients.
	
	Args: none.

	Returns: none.
--------------------------------------------------------------------------------------------------------------------------------------
TestLastHundred.m --> function

	Takes in a cell containing shape descriptors for multiple objects, a cell of labels for each set of descriptors, a number of neighbors
	to check, and a number of descriptors to use, and trains a KNN model on each of these things.
	The KNN model is trained on all but the last 100 training data points. Once the KNN model is trained using the specified number of 
	neighbors and number of descriptors, the 100 training data points that were held out are passed to the model, and the model's accuracy
	for that particular set of data is determined.
	
	Args: inDescriptors - A cell of cells containing descriptors for a particular descriptor method. Each outer level cell represents an
						  "index" of a particular object, the next layer of cells contain descriptors created from Fourier coefficients
						  whose corresponding frequencies increase with index.
		  labels - A cell of integer labels referring to the name of the handwritten digit whose descriptors are contained in the 
				   "inDescriptors" variable.
		  numDescriptors - The total number of descriptors to use to try and characterize a certain shape with KNN.
		  numNeighbors - The total number of neighbors that will be allowed to weigh-in and classify a new piece of data in KNN space.
		  
	Returns: result - A histogram of the number of failed classifications arranged by label.
					  'result[1]' represents the number of times the handwritten digit '0' was misclassified.
--------------------------------------------------------------------------------------------------------------------------------------
TraverseOuterContour.m --> function

	Given a bilevel image (0 as BG, anything > 0 as FG), this method attempts to find a contour that encloses an object's entire outer
	surface. This function encounters errors with "thin" objects, e.g. objects that are only one or two pixels thick in any spot.
	For your convenience, the helper file "helper.m" directly demonstrates the failure that this function can experience. Errors can
	occur while tracking the object's contour; if the number of main loop iterations exceeds the total number of pixels in the image, 
	the process is terminated and an error is returned. The reasoning behind this is that it would be impossible to have a contour in
	a digital, bilevel image with a number of contour points equal to the total number of pixels in the image.
	
	Args: I - A bilevel image.
	
	Returns: contourPixels - A one-dimensional cell containing tuples of coordinates that, when accessed in sequential order, track the
							 contour of a given object.
			 fail - A flag that indicates an error occurred while trying to track the object's contour. 
--------------------------------------------------------------------------------------------------------------------------------------

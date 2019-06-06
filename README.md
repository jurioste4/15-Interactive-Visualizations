# Belly Button Biodiversity

I was tasked with creating an   interactive dashboard to exploring belly button Biodiversity : http://robdunnlab.com/projects/belly-button-biodiversity/   data set. 
Data set was stored in db folder bellybutton.sqlite 
I used Python flask app.py: using SQAlchemy flask to connect and display the data 
I then created used JavaScript located in file folder static / js/ app.js 
I used Plotly.js and some D3 to create the plots and dropdown menu 
Create a PIE chart that uses data from your samples route (/samples/<sample>) to display the top 10 samples.
I Use sample_values as the values for the PIE chart 
Then otu_ids as the labels for the pie chart
And otu_labels as the hovertext for the chart

Then used route /metadata/<sample> to display the metadata JSON Object on the page 


Display each key/value pair from the metadata JSON object somewhere on the page

then I Deploy  Flask app to Heroku.  https://visual-jurioste4.herokuapp.com/

#################################################
# Database Setup
#################################################
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///db/bellybutton.sqlite"


db = SQLAlchemy(app)

# reflect an existing database into a new model

Base = automap_base()

# reflect the tables

Base.prepare(db.engine, reflect=True)

# Save references to each table

Samples_Metadata = Base.classes.sample_metadata

Samples = Base.classes.samples



#### then set up the routes two of them.

First one for the index.html to render 

@app.route("/")

def index():

"""Return the homepage."""

return render_template("index.html")

then create a names rout 

@app.route("/names")

def names():

"""Return a list of sample names."""

# Use Pandas to perform the sql query


    stmt = db.session.query(Samples).statement

df = pd.read_sql_query(stmt, db.session.bind)


  # Return a list of the column names (sample names)
   
    return jsonify(list(df.columns)[2:])

creating next app.route 

@app.route("/metadata/<sample>")

def sample_metadata(sample):

"""Return the MetaData for a given sample."""

sel = [

Samples_Metadata.sample,

Samples_Metadata.ETHNICITY,

Samples_Metadata.GENDER,

Samples_Metadata.AGE,

Samples_Metadata.LOCATION,

Samples_Metadata.BBTYPE,

Samples_Metadata.WFREQ,
                    
                    ]
    results = db.session.query(*sel).filter(Samples_Metadata.sample == sample).all()

using the above info for the next route 


@app.route("/samples/<sample>")

def samples(sample):

"""Return `otu_ids`, `otu_labels`,and `sample_values`."""

stmt = db.session.query(Samples).statement

df = pd.read_sql_query(stmt, db.session.bind)
 
 # Filter the data based on the sample number and
 
   # only keep rows with values above 1
    

sample_data = df.loc[df[sample] > 1, ["otu_id", "otu_label", sample]]

# Format the data to send as jsonw
    
    data = {

"otu_ids": sample_data.otu_id.values.tolist(),

"sample_values": sample_data[sample].values.tolist(),

"otu_labels": sample_data.otu_label.tolist(),

}

return jsonify(data)


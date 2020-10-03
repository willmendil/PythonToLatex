<article>

    <h1>Native Looking matplotlib Plots in LaTeX</h1>

    
        <aside>
    <ul>
        <li>
            <time class="post-date" datetime="2014-04-15T00:00:00Z">Apr 15, 2014</time>
        </li>
        
        

        
        <li>
            <em>
                
                    
                    <a href="http://bkanuka.com/tags/latex/">#latex</a>
                
                    , 
                    <a href="http://bkanuka.com/tags/matplotlib/">#matplotlib</a>
                
                    , 
                    <a href="http://bkanuka.com/tags/latex/">#latex</a>
                
                    , 
                    <a href="http://bkanuka.com/tags/tricks/">#tricks</a>
                
                    , 
                    <a href="http://bkanuka.com/tags/script/">#script</a>
                
            </em>
        </li>
        

        <li>2 min read</li>
    </ul>
</aside>
    

    

<p>I write most of my math/numerical analysis scripts in Python, and I tend to use <a href="http://matplotlib.org/">matplotlib</a> for plotting.
When including a matplotlib plot in LaTeX I got the highest quality results by saving the plot as a PDF and using <code>\includegraphics{plot.pdf}</code> in LaTeX.
However, it bothered me that the plot had different fonts and font sizes than the rest of the document.
Here’s how I fixed that.</p>

<h2 id="figure-width">Figure Width</h2>

<p>I always choose the size of my plots as a percentage of the text width.
For example <code>width=0.6\textwidth</code>.
This allows me to use <code>0.3\textwidth</code> for images that are going to be side-by-side and not worry about absolute sizes.
We want matplotlib to output the right size plot so we need to find what exactly the <code>textwidth</code> is and tell matplotlib.
Do this by writing <code>\the\textwidth</code> inside your LaTeX document (inside the document, <em>not</em> the preamble) and running it through <code>pdflatex</code> or whatever LaTeX engine you use.
You’ll find that LaTeX will replace the command with some number.
Record this number.</p>

<h2 id="generate-figures">Generate Figures</h2>

<p>For every LaTeX document that has plots, I write a script <code>figures.py</code> which creates all the plots.
Copy the following script into <code>figures.py</code> and save it into the same folder as your LaTeX document.
Replace <code>fig_width_pt</code> with whatever number you got from above.</p>

<script src="//gist.github.com/bkanuka/10796230.js"></script><link rel="stylesheet" href="https://github.githubassets.com/assets/gist-embed-fd43f22140a6ad2cc9d0aa1f169a01f3.css"><div id="gist10796230" class="gist">
    <div class="gist-file">
      <div class="gist-data">
        <div class="js-gist-file-update-container js-task-list-container file-box">
  <div id="file-figures-template-py" class="file my-2">
    

  <div itemprop="text" class="Box-body p-0 blob-wrapper data type-python  ">
      
<code>
import numpy as np
import matplotlib as mpl
mpl.use('pgf')

def figsize(scale):
    fig_width_pt = 469.755                          # Get this from LaTeX using \the\textwidth
    inches_per_pt = 1.0/72.27                       # Convert pt to inch
    golden_mean = (np.sqrt(5.0)-1.0)/2.0            # Aesthetic ratio (you could change this)
    fig_width = fig_width_pt*inches_per_pt*scale    # width in inches
    fig_height = fig_width*golden_mean              # height in inches
    fig_size = [fig_width,fig_height]
    return fig_size

pgf_with_latex = {                      # setup matplotlib to use latex for output
    "pgf.texsystem": "pdflatex",        # change this if using xetex or lautex
    "text.usetex": True,                # use LaTeX to write all text
    "font.family": "serif",
    "font.serif": [],                   # blank entries should cause plots to inherit fonts from the document
    "font.sans-serif": [],
    "font.monospace": [],
    "axes.labelsize": 10,               # LaTeX default is 10pt font.
    "font.size": 10,
    "legend.fontsize": 8,               # Make the legend/label fonts a little smaller
    "xtick.labelsize": 8,
    "ytick.labelsize": 8,
    "figure.figsize": figsize(0.9),     # default fig size of 0.9 textwidth
    "pgf.preamble": [
        r"\usepackage[utf8x]{inputenc}",    # use utf8 fonts becasue your computer can handle it :)
        r"\usepackage[T1]{fontenc}",        # plots will be generated using this preamble
        ]
    }
mpl.rcParams.update(pgf_with_latex)

import matplotlib.pyplot as plt

# I make my own newfig and savefig functions
def newfig(width):
    plt.clf()
    fig = plt.figure(figsize=figsize(width))
    ax = fig.add_subplot(111)
    return fig, ax

def savefig(filename):
    plt.savefig('{}.pgf'.format(filename), bbox_inches='tight')
    plt.savefig('{}.pdf'.format(filename), bbox_inches='tight')


# Simple plot
fig, ax  = newfig(0.6)

def ema(y, a):
    s = []
    s.append(y[0])
    for t in range(1, len(y)):
        s.append(a * y[t] + (1-a) * s[t-1])
    return np.array(s)
    
y = [0]*200
y.extend([20]*(1000-len(y)))
s = ema(y, 0.01)

ax.plot(s)
ax.set_xlabel('X Label')
ax.set_ylabel('EMA')

savefig('ema')
</code>


  </div>

  </div>
</div>

      </div>
      <div class="gist-meta">
        <a href="https://gist.github.com/bkanuka/10796230/raw/24b594c22f8bbe4c0a11a8384e50a4bbe08e31c4/figures-template.py" style="float:right">view raw</a>
        <a href="https://gist.github.com/bkanuka/10796230#file-figures-template-py">figures-template.py</a>
        hosted with ❤ by <a href="https://github.com">GitHub</a>
      </div>
    </div>
</div>


<p>You <em>must</em> <code>import matplotlib</code> and make any rc changes before importing <code>matplotlib.pyplot</code>.
matplotlib expresses sizes in inches, while LaTeX likes sizes to be in pt, so the first part of this script sets up sizes in matplotlib properly.
The figure height is determined by the golden ratio, which is highly aesthetic ratio (it’s a good default).</p>

<h2 id="latex">LaTeX</h2>

<p>Running the above with <code>python figures.py</code> produces two files: <code>ema.pdf</code> and <code>ema.pgf</code>.
The PDF file is used just to have a stand-alone version of the plot and make sure everything looks right.</p>

<p>To incorporate the plot into LaTeX, put <code>\usepackage{pgf}</code> in the preamble and insert using <code>\input{ema.pgf}</code>.
For example:</p>

<div class="highlight"><pre style="color:#f8f8f2;background-color:#272822;-moz-tab-size:4;-o-tab-size:4;tab-size:4"><code class="language-latex" data-lang="latex"><span style="color:#66d9ef">\documentclass</span>{article}
<span style="color:#66d9ef">\usepackage</span>{pgf}

<span style="color:#66d9ef">\begin</span>{document}

<span style="color:#66d9ef">\begin</span>{figure}
    <span style="color:#66d9ef">\caption</span>{A simple EMA plot.<span style="color:#66d9ef">\label</span>{fig:ema1}}
    <span style="color:#66d9ef">\centering</span>
    <span style="color:#66d9ef">\input</span>{ema.pgf}
<span style="color:#66d9ef">\end</span>{figure}

<span style="color:#66d9ef">\end</span>{document}</code></pre></div>

<p><img src="http://bkanuka.com/images/ema.png" alt="ema plot example"></p>

<hr>

<p><em>Thank you Dan for the suggested changes to my script</em></p>


</article>

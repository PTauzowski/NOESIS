id: CantileverTipDeflection

type: Block

computes:
  - TipDisplacement

requires:
  - Force
  - BeamLength
  - YoungModulus
  - SecondMomentOfArea

assumptions:
  - LinearElasticity
  - SmallDeflection
  - EulerBernoulliTheory

validation:
  - AnalyticalSolution

implementations:
  - Python
  - MATLAB